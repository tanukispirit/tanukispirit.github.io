---
  GNU nano 5.4                                     2023-03-24-ncdocker.md                                               ---
title: TubeArchivist with Docker Compose
date: 2023-03-09 12:00:00 -500
categories: [lab]
tags: [pi, tubearchivist, docker, docker-compose]
---

# Install TubeArchivist with Docker Compose
## Steps to install on Raspberry Pi (64bit)

```Dockerfile

version: '3.3'

services:
  tubearchivist:
    container_name: tubearchivist
    restart: unless-stopped
    image: bbilly1/tubearchivist
    ports:
      - 8000:8000  # change the first port number if you need to xxxx:8000
    volumes:
      - /path/to/your/Downloads:/youtube
      - /path/to/your/cache:/cache
    environment:
      - ES_URL=http://archivist-es:9200     # needs protocol e.g. http and port
      - REDIS_HOST=archivist-redis          # don't add protocol
      - HOST_UID=1000
      - HOST_GID=1000
      - TA_HOST=Server_Address         # set your host name
      - TA_USERNAME=Your_custom_name          # your initial TA credentials
      - TA_PASSWORD=Change-me             # your initial TA credentials
      - ELASTIC_PASSWORD=Change-me        # set password for Elasticsearch
      - TZ=America/Chicago                 # set your time zone
    depends_on:
      - archivist-es
      - archivist-redis
  archivist-redis:
    image: bbilly1/rejson:latest
    container_name: archivist-redis
    restart: unless-stopped
    expose:
      - "6379"                  # don't change
    volumes:
      - /path/to/your/redis:/data
    depends_on:
      - archivist-es
  archivist-es:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.7.0-arm64         # This one is for Raspberry Pi's
    container_name: archivist-es
    restart: unless-stopped
    environment:
      - "ELASTIC_PASSWORD=Change-me       # matching Elasticsearch password
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "xpack.security.enabled=true"
      - "discovery.type=single-node"
      - "path.repo=/usr/share/elasticsearch/data/snapshot"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - /path/to/your/data    # check for permission error when using bind>    expose:
      - "9200"                # don't change

```

```bash
docker compose up -d
```
