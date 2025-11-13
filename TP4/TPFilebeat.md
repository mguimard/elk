# TP 4 : Ingestion des logs Apache avec Filebeat

L’objectif de cet exercice est de configurer Filebeat pour ingérer des logs Apache depuis un serveur de production, créer un tableau de bord personnalisé et écrire des requêtes d’agrégation.

## Les données

Le fichier de données est nommé **access.log**. Analysez les premières lignes.

## Configuration : Installation de Filebeat

Téléchargez Filebeat et extrayez-le dans votre répertoire utilisateur.

[https://www.elastic.co/downloads/beats/filebeat](https://www.elastic.co/downloads/beats/filebeat)

Modifiez le fichier `filebeat.yml` pour ajouter la configuration de votre ElasticSearch et Kibana :

```yaml
setup.kibana:
  host: "localhost:5601"

output.elasticsearch:
  hosts: ["localhost:9200"]
  preset: balanced
  protocol: "https"
  username: "elastic"
  password: "CHANGEME"
  ssl.verification_mode: "none"
```

Activez le module Apache :

```bash
./filebeat modules enable apache
```

Configurez le module Apache :

```yaml
# file: modules.d/apache.yml
- module: apache
  access:
    enabled: true
    var.paths:
      - /path/to/access.log

  # Error logs
  error:
    enabled: false
~              
```

Générez les dashboards et pipelines :

```bash
./filebeat setup dashboards
```

Inspectez le pipeline d’ingestion Apache depuis DevTools :

```bash
GET _ingest/pipeline/*apache*
```

Inspectez les data streams :

```bash
GET _data_stream
```

## Ingestion et Monitoring

Activez le monitoring dans Kibana → Stack Monitoring. Identifiez les écrans de monitoring principaux.

Ensuite, démarrez l’ingestion des données :

```bash
./filebeat
```

Surveillez le processus d’ingestion (plusieurs millions de lignes).

Une fois l’ingestion terminée, identifiez l’index contenant les logs. Accédez au tableau de bord Apache prédéfini et étudiez-le.

## Tableau de bord personnalisé

Créez un tableau de bord personnalisé affichant :

* Un graphique temporel montrant les requêtes **200 OK** et **404 NOT FOUND**
* Un graphique type camembert montrant les pays les plus fréquents
* Un graphique temporel montrant la tendance d’utilisation des navigateurs web
* Une carte du monde affichant la localisation des requêtes

Ajoutez d'autres panneaux si vous le souhaitez.

## Requêtes d’agrégation

Écrivez les requêtes d’agrégation suivantes :

* Moyenne des octets envoyés aux clients
* Liste des différents pays
* Liste des différents codes de statut HTTP et leur nombre
* Moyenne des octets envoyés par IP
