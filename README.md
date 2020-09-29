# Alfresco Search Services with Replication

Deployment template based in official [Docker Composition](https://github.com/Alfresco/acs-community-deployment/tree/master/docker-compose) using SOLR Replication with two nodes in Master / Slave configuration.

This configuration provides a dedicated node for *tracking* named `solrmaster` and a dedicated node for *searching* name `solrslave`. Despite this feature is available for both Enterprise and Community versions, documentation is available at [SOLR Replication](https://docs.alfresco.com/search-enterprise/concepts/solr-replication.html).

SOLR Replication with Master / Slave configuration is recommended to fulfil some of following requirements:

* Splits read and write load and operations in SOLR Slaves and SOLR Master
* Load distribution for search queries
* High availability for searching

You should review volumes, configuration, modules & tuning parameters before using this composition in **Production** environments.

## Source Images

* [alfresco-content-repository-community:6.2.0-ga](https://hub.docker.com/r/alfresco/alfresco-content-repository-community)
* [alfresco-share:6.2.0](https://hub.docker.com/r/alfresco/alfresco-share)
* [alfresco-search-services:2.0.0.1](https://hub.docker.com/r/alfresco/alfresco-search-services)
* [postgres:11.4](https://hub.docker.com/_/postgres)
* [nginx:stable-alpine](https://hub.docker.com/_/nginx)

## Project structure

```
.
├── .env
├── config
│   ├── nginx.conf
│   └── nginx.htpasswd
├── docker-compose.yml
└── replication-none
    └── Dockerfile
```

* `.env` includes Docker environment variables to set Docker Image release numbers
* `config` folder includes configuration for HTTP Proxy based in NGINX
* `docker-compose.yml` is a Docker Compose template to use ACS Community with SOLR Replication
* `replication-none` folder includes a customised Dockerfile to configure SOLR Docker Image in master and slave mode

## SOLR Considerations

Alfresco SOLR API has been protected to be accessed from outside Docker network, as using HTTP allows unauthenticated requests.

```
    # Protect access to SOLR APIs
    location ~ ^(/.*/service/api/solr/.*)$ {return 403;}
    location ~ ^(/.*/s/api/solr/.*)$ {return 403;}
    location ~ ^(/.*/wcservice/api/solr/.*)$ {return 403;}
    location ~ ^(/.*/wcs/api/solr/.*)$ {return 403;}

    location ~ ^(/.*/proxy/alfresco/api/solr/.*)$ {return 403 ;}
    location ~ ^(/.*/-default-/proxy/alfresco/api/.*)$ {return 403;}
```

SOLR Web Console access has been protected with username/password (admin/admin).


# How to use this composition

## Start Docker

Start docker and check the ports are correctly bound.

```bash
$ docker-compose up -d
$ docker ps --format '{{.Names}}\t{{.Image}}\t{{.Ports}}'
alfresco_1    alfresco/alfresco-content-repository-community:6.2.0-ga 8080/tcp
share_1       alfresco/alfresco-share:6.2.0                           8000/tcp, 8080/tcp
solr6slave_1  solr6slave                                              8983/tcp
solr6master_1 solr6master                                             8983/tcp
proxy_1       nginx:stable-alpine                                     0.0.0.0:80->80/tcp
postgres_1    postgres:11.4                                           0.0.0.0:5432->5432/tcp
```

### Viewing System Logs

You can view the system logs by issuing the following.

```bash
$ docker-compose logs -f
```

Logs for every service are also available at `logs` folder.

## Access

Use the following username/password combination to login.

 - User: admin
 - Password: admin

Alfresco and related web applications can be accessed from the below URIs when the servers have started.

```
http://localhost/alfresco      - Alfresco Repository
http://localhost/solrmaster    - Alfresco Search Services (Master, tracking)
http://localhost/solr          - Alfresco Search Services (Slave, searching)
http://localhost/share         - Alfresco Share
```
