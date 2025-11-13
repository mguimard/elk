# TP 3 : Logstash

1. Création index

```json
PUT sncf
{
  "mappings": {
    "properties": {
      "date_controle": {
        "type": "date"
      },
      "gare": {
        "type": "completion",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "code_uic": {
        "type": "keyword"
      },
      "problemes": {
        "type": "integer"
      },
      "observations": {
        "type": "integer"
      }
    }
  }
}
```

2. Ajout champs virtuels

```json
PUT sncf/_mapping
{
  "runtime": {
      "taux_conformite": {
        "type": "double",
        "script": {
          "source": """
          if( doc['observations'].size()!=0 && doc['problemes'].size() != 0 && doc['observations'].value > 0 && doc['problemes'].value > 0)
            emit((1.0 - 1.0 * doc['problemes'].value / doc['observations'].value) * 100.0);
          else
            emit(0);
          """
        }
      },
      "hour_of_day": {
        "type": "double",
        "script": {
          "source": """
          if(doc['date_controle'].size()!=0 )
            emit(doc['date_controle'].value.getHour());
          else
            emit(0);
          """
        }
      },
      "day_of_week": {
        "type": "keyword",
        "script": {
          "source": """
          if(doc['date_controle'].size() == 0)
            emit('NA');
          else
          emit(doc['date_controle'].value.dayOfWeekEnum.getDisplayName(TextStyle.FULL, Locale.ROOT));
          """
        }
      }
    }
}
```

3. Configuration logstash

```
input {
    file {
        path => "/partage/sncf-non-conformites.csv"
        start_position => "beginning"
        sincedb_path => "/dev/null"
    }
}

filter {
    csv {
        separator => ";"
        columns => ["date_controle","code_uic","gare","observations","problemes"]
    }
}

output {
   elasticsearch {
        action => "index"
        index => "sncf"
        hosts => ["https://localhost:9200"]
        ssl_verification_mode => "none"
        password => "CHANGEME"
        user => "elastic"
    }
    stdout { }
}
```

4. Insérer les données

```bash
bin/logstash -f /path/to/config-file
```

5. Dashboard

Créer une dataview, champ temporel : date_controle.

Faire parler les données sous forme de dashboard.
