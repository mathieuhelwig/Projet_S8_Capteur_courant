---
layout: home
title: Réalisation (montage/câblage)
parent: Réalisation et programmation de l'objet connecté
grand_parent: Conception et prototypage
nav_order: 1
---

# Réalisation

<p>
    Dans cette première partie nous présentons les différents composants de notre objet connecté et la manière dont ils seront raccordés ensemble.<br>
</p>

# 1. Présentation du capteur de courant SC-013

<p>
    Le capteur de courant SC-013 est un capteur de courant non invasif, c'est-à-dire qu'il peut être installé sur le phase du circuit à mesurer sans avoir à intervenir sur celle-ci. Aucune intervention sur l'installation électrique n'est donc nécessaire.<br>
    En effet, il se présente sous la forme d'un aimant coupé en 2 sur lequel est enroulé un fil électrique.
    L'aimant coupé en 2 permet d'ouvrir le SC-013 et de le refermer autour du conducteur dans lequel passe le courant à mesurer. Dans le fil secondaire autour de l'aimant circule alors un courant proportionnel au courant à mesurer circulant dans le fil conducteur primaire.<br>
    En ce sens, son fonctionnement est proche de celui d'un transformateur et repose sur le principe des courants induits.<br>
</p>

<div style="display: inline-block; width: 1030px; max-width: 100%; border: 1px solid black;">
<img
    style="display: inline-block; 
            width: 33%;
            float: left;"
src="../images/SC-013_non_invasive_current_detector.jpg"
alt = "SC-013_non_invasive_current_detector">
<img
    style="display: inline-block; 
            width: 33%;
            float: center;"
src="../images/SC-013_principe_de_fonctionnement.jpg"
alt = "SC-013_principe_de_fonctionnement">
<img
    style="display: inline-block; 
            width: 33%;
            float: right;"
src="../images/SC-013_schema_montage_autour_fil_conducteur.jpg"
alt = "SC-013_schema_montage_autour_fil_conducteur"></div>

<p style="text-align: center;"><em>Présentation du SC-013: capteur non invasif - principe de fonctionnement - montage sur le fil conducteur</em></p>

<p>
    Il existe différentes références du SC-013, avec ou sans résistance de charge. Pour simplifier le montage, nous avons selectionné la référence SC-013-050, équipé d'une résistance de charge intégrée et permettant de mesurer des courants primaires jusqu'à 50A (soit une puissance jusqu'à 11500W).<br>
    On aura donc en sortie du SC-013-050 une tension de 1V RMS (signal analogique), correspondant aux 50A dans le fil conducteur.<br>
    La tension RMS (signifiant <em>Root Mean Square</em>) représente une mesure de la tension crête-à-crête (ou <em>peak-to-peak</em> en anglais). La relation entre la valeur RMS et la valeur crête-à-crête d'une tension est la suivante: 
</p>

<img
    style="display: block; 
           margin-left: auto;
           margin-right: auto;"
src="../images/V_rms_ptp.jpg"
alt = "relation entre tension RMS et crete a crete">

<p>
    C'est pourquoi, dans le cas du SC-013-050 avec une tension de sortie de 1V RMS, la tension de crête sera de +/- 1.414V et la tension crête-à-crête sera de 2.828V. Nous verrons un peu plus loin que ces valeurs sont importantes dans l'écriture du programme pour l'Arduino Uno.
</p>

# 2. Présentation du convertisseur analogique-numérique ADS1115

<p>
    Le convertisseur analogique-numérique ADS-1115 permet de convertir une mesure analogique (signal continu) en une mesure numérique. Il prend en charge le protocole de communication I2C, ce qui permet de raccorder jusqu'à 4 ADS-1115 sur la même ligne de communication I2C. C'est un convertisseur 16 bits, c'est-à-dire que le signal analogique mesuré sera converti en une valeur sur 2 octets. Il est équipé d'un comparateur programmable, c'est-à-dire qu'il peut convertir une difference de potentiel (une tension donc) entre 2 de ses entrées. Cela nous interesse particulièrement car nous pourrons ainsi convertir la tension (proportionnelle au courant mesuré) delivrée par le SC-013-050 en signal numérique.<br>
    En ce qui concerne le raccordement entre le SC-013 et l'ADS1115, cela se fera grâce à un composant de raccordement "jack vers breadboard" cité dans la partie "Etudes et choix techniques".<br>
    Ci-dessous sont présentés 3 schémas qui permettent de comprendre et de reproduire le raccordement entre la sortie jack d'un SC-013 et un ADS1115 jusqu'aux broches de l'Arduino Uno.
</p>

<div style="display: inline-block; width: 1030px; max-width: 100%; border: 1px solid black;">
<img
    style="display: inline-block; 
            width: 35%;
            float: left;"
src="../images/jack_corresponding_on_SC-013.jpg"
alt = "jack_corresponding_on_SC-013">
<img
    style="display: inline-block; 
            width: 45%;
            float: center;"
src="../images/SC-013_mounting_scheme_on_ADS1115.jpg"
alt = "SC-013_mounting_scheme_on_ADS1115">
<img
    style="display: inline-block; 
            width: 20%;
            float: right;"
src="../images/ADS1115_mounting_scheme_on_Arduino.jpg"
alt = "ADS1115_mounting_scheme_on_Arduino"></div>

<p style="text-align: center;"><em>Schémas de raccordement entre un SC-013 et un ADS1115 jusqu'aux broches de l'Arduino Uno</em></p>

<p>
    Pour notre objet connecté, nous avons besoin de raccorder 3 SC-013 et donc 3 ADS1115 sur l'Arduino. Pour cela, nous allons utiliser le bus I2C des ADS1115 et de l'Arduino. Ci-dessous est présenté le schéma de raccordement de 4 ADS1115 (nombre maximum d'ADS1115) sur un microcontroleur prenant en charge le bus I2C (extrait de la documentation technique de l'ADS1115, page 34) :
</p>

<img
    style="display: block;
            width: 90%;
            margin-left: auto;
            margin-right: auto;"
src="../images/schema_raccordement_4_ADS1115_sur_microcontroleur_bus_I2C.jpg"
alt = "schema_raccordement_4_ADS1115_sur_microcontroleur_bus_I2C">

<p style="text-align: center;"><em>Schéma de raccordement de 4 ADS1115 sur un microcontroleur prenant en charge le bus I2C</em></p>

<p>
    Dans la documentation technique de l'ADS1115, il est pécisé dans la partie "Programming / I2C Address Selection" (partie 9.5.1.1 page 23) que les 3 adresses correspondant à la connexion des pins d'adresses sur GND, VDD et SCL doivent être utilisées en premier. C'est ce que nous respecterons pour notre objet connecté.
</p>

# 3. La carte programmable Arduino Uno et son shield W5100


# 4. Schéma du montage
