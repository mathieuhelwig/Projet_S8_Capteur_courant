---
layout: home
nav_order: 3
title: Études et choix techniques
---

# Études et choix techniques

Nous détaillons dans cette partie les choix techniques concernant les 3 étapes de conception et de réalisation du prototype:
## 1. Réalisation et programmation de l'objet connecté, basé sur:
&emsp;&emsp;- 3 capteurs de courants non invasifs [SC-013-050](https://www.amazon.fr/dp/B07MY3NNFR?psc=1&ref=ppx_yo2ov_dt_b_product_details),\
&emsp;&emsp;- 3 modules de conversion analogique-numérique [ADS1115](https://www.amazon.fr/dp/B07QHWLTTS?ref=ppx_yo2ov_dt_b_product_details&th=1),\
&emsp;&emsp;- 1 [carte Arduino Uno](https://www.amazon.fr/dp/B01JD2Z5XW?psc=1&ref=ppx_yo2ov_dt_b_product_details),\
&emsp;&emsp;- 1 [shield ethernet W5100](https://www.amazon.fr/dp/B07XYYJXM3?psc=1&ref=ppx_yo2ov_dt_b_product_details)\
&emsp;&emsp;- des [composants de raccordement](https://www.amazon.fr/dp/B0B5D3YBLZ?psc=1&ref=ppx_yo2ov_dt_b_product_details) pour les jacks des SC-013,\
&emsp;&emsp;- quelques [fils suplémentaires](https://www.amazon.fr/dp/B07KWBHL11?psc=1&ref=ppx_yo2ov_dt_b_product_details) de bonne qualité, ce n'est pas un mauvais investissement...
## 2. Montage d'une VM sur un serveur pour:
&emsp;&emsp;- héberger Portainer (gestionnaire de conteneurs type Docker),\
&emsp;&emsp;- monter dans une même stack:\
&emsp;&emsp;&emsp;- une image Orion et MongoDB pour la gestion des objets Fiware,\
&emsp;&emsp;&emsp;- une image Node-Red pour la gestion des flux et l'affichage des données sur un dashboard,\
&emsp;&emsp;&emsp;- une image MariaDB pour le stockage en historique des objets Fiware,\
&emsp;&emsp;&emsp;- une image d'un serveur MQTT tel que Mosquitto pour la récupération des données transmises par l'objet connecté.
## 3. Développement des flux Node-Red permettant:
&emsp;&emsp;- la récupération des données de l'objet connecté,\
&emsp;&emsp;- la création et l'enregistrement dans MongoDB des objets Fiware, ainsi que la création d'abonnements pour remonter d'éventuelles alertes sur le dashboard,\
&emsp;&emsp;- l'enregistrement historisé dans MariaDB des données transmises par l'objet connecté,\
&emsp;&emsp;- l'affichage des données historisées et instantanées dans un graphique sur le dashboard Node-Red.