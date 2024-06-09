---
layout: default
title: Déploiement de la VM
parent: Conception et prototypage
nav_order: 2
---

# 2. Montage d'une VM sur un serveur

## 2.1 Caractéristiques de la VM

&emsp;- Système d'exploitation:\
&emsp;- Nombre de coeurs CPU:\
&emsp;- Mémoire vive:\
&emsp;- Espace disque:

## 2.2 Caractéristiques du conteneur

&emsp;- Les différentes images à intégrer:

&emsp;- Fichier .yml de déploiement de la stack:

```
### Création d'un environnement persistant mongoDB-Orion-NodeRed-MariaDB-MQTT Mosquitto ###

# 1. créer préalablement 4 dossiers ("mongo", "nodered", "mosquitto", "mariadb") dans le répertoire courant pour le stockage des données persistantes
# 2. mapper les volumes non persistants créés lors du déploiement de la stack vers les volumes créés à l'étape 1.
# 3. s'assurer que les images sont sur un même réseau interne et ont des adresses IP internes (sur port 1026 pour Orion, sur port 1880 pour Node-Red, sur port 1883 pour Mosquitto, sur port 3306 pour MariaDB)
# 4. configurer Mosquitto (fichier config et passwd, voir https://itbacon.com/2023/08/01/installing-mosquitto-mqtt-in-portainer/)
# 5. configurer MariaDB en créant la BDD et les tables necessaires (1 temporaire pour moyenne des puissances sur 1 minute, 1 pour enregistrement toutes les minutes)
version: "3.9"
services:
  mongo:
    container_name: "mongoDB"
    image: mongo:4.4
    command: --nojournal
    networks:
      - internal
    volumes:
      - ./mongo:/data/db
  orion:
    container_name: "orion"
    image: fiware/orion
    ports:
      - "1026:1026"
    networks:
      - internal
    command: -dbhost mongo
  nodered:
    container_name: "node-red"
    image: nodered/node-red
    ports:
      - "1880:1880"
    networks:
      - internal
    volumes:
      - ./nodered:/data
  mosquitto:
    container_name: "mosquitto"
    restart: "unless-stopped"
    image: "eclipse-mosquitto"
    networks:
      - internal
    ports:
      - "1883:1883/tcp"
    volumes:
      - ./mosquitto:/mosquitto/config
      - ./mosquitto:/mosquitto/data
      - ./mosquitto:/mosquitto/log
  mariadb:
    container_name: "mariadb"
    image: mariadb
    restart: always
    networks:
      - internal
    ports:
      - "3306:3306/tcp"
    volumes:
      - ./mariadb:/data
    environment:
      MARIADB_ROOT_PASSWORD: example #à changer

networks:
  internal:
```


&emsp;- Manipulations postérieures au déploiement de la stack:\
&emsp;&emsp;- volumes à mapper pour la persistance des données:\
&emsp;&emsp;- configuration serveur Mosquitto:

## 2.3 Résultat de la stack complète montée dans Portainer

<img
    style="display: block; 
           margin-left: auto;
           margin-right: auto;
           width: 100%;"
src="../images/stack_portainer.jpg"
alt="stack Portainer">
<p style="text-align: center;"><em>Le résultat de la stack avec les différentes images déployées dans Portainer</em></p>
