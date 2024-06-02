---
layout: default
parent: Premiers pas sur le template
nav_order: 5
title: Modifier Votre Site Jekyll sur GitHub - Avancé
---
# Modifier Votre Site Jekyll sur GitHub - Avancé

Il est interessant de pouvoir construire localement son site sur son ordinateur avant de l'envoyer sur Github. Cela évite les multiples commits, et cela accelère le développement.

1. INDEX
{:toc}

Ce guide vous montrera étape par étape comment configurer votre environnement Windows pour construire un site de documentation avec Jekyll en utilisant Visual Studio Code (VSCode).

Une fois cela effectué, vous pourrez voir votre site localement dans votre navigateur en utilisant l'URL suivante : [http://127.0.0.1:4000](http://127.0.0.1:4000).

A noter que si vous le souhaitez, vous pouvez également utiliser l'envirronement WSL pour construire localement votre site. Mais cette méthode, bien que similaire, demande un peu plus de connaissances et n'est pas décrite ici. 

## Sur Windows

### Étape 1 : Installer Visual Studio Code

1. Téléchargez et installez Visual Studio Code depuis [le site officiel](https://code.visualstudio.com/).
2. Suivez les instructions d'installation et lancez VSCode une fois l'installation terminée.

### Étape 2 : Installer Ruby

1. Téléchargez et installez Ruby via [RubyInstaller](https://rubyinstaller.org/). Choisissez une version adaptée à votre système (x86 ou x64).
2. Pendant l'installation, **assurez-vous de cocher l'option** pour ajouter Ruby aux variables d'environnement de Windows.
3. Après l'installation, redémarrez votre ordinateur pour que les changements prennent effet.

### Étape 3 : Vérifier l'Installation de Ruby

1. Ouvrez VSCode.
2. Ouvrez un nouveau terminal dans VSCode (Terminal -> Nouveau Terminal).
3. Tapez `ruby -v` pour vérifier que Ruby est correctement installé. Vous devriez voir la version de Ruby s'afficher.

### Étape 4 : Cloner le Répertoire de Documentation

1. Dans VSCode, allez dans l'onglet "Source Control" (icône de branches en bas à gauche).
2. Cliquez sur "Clone Repository".
3. Entrez l'URL de votre répertoire GitHub contenant la documentation Jekyll.
4. Choisissez un emplacement sur votre ordinateur pour cloner le répertoire et suivez les instructions.

{: .note-title }
> A noter
>
> Cette partie peut être exécutée avec Github Desktop. Dans ce cas, une fois le repo cloné, vous ouvrirez le dossier dans VSCode.

### Étape 5 : Ouvrir le Dossier de Documentation

1. Dans VSCode, ouvrez le dossier cloné (Fichier -> Ouvrir Dossier).
2. Naviguez jusqu'au dossier `docs/`.
3. Faites un clic droit sur le dossier `docs/` et sélectionnez "Ouvrir dans le Terminal".

### Étape 6 : Installation des Dépendances

Dans le terminal ouvert :

1. Exécutez `gem install bundler` pour installer Bundler.
2. Exécutez `bundle install` pour installer toutes les dépendances requises par Jekyll et votre projet.

### Étape 7 : Construire et Servir le Site

1. Dans le terminal, exécutez `bundle exec jekyll serve`.
2. Ouvrez un navigateur web et allez à `http://localhost:4000` pour voir votre site Jekyll en action.

---

## Sur Linux (non testé)

### Étape 1 : Installer Visual Studio Code
{: .no_toc }

1. Téléchargez et installez Visual Studio Code depuis [le site officiel](https://code.visualstudio.com/).
2. Vous pouvez également utiliser votre gestionnaire de paquets pour l'installer, par exemple `sudo apt install code` sur les distributions basées sur Debian/Ubuntu.

### Étape 2 : Installer Ruby
{: .no_toc }

1. Ouvrez un terminal.
2. Installez Ruby via votre gestionnaire de paquets, par exemple `sudo apt install ruby-full` sur les distributions basées sur Debian/Ubuntu.
3. Vérifiez l'installation avec `ruby -v`.

**Étape 3 : similaires à celles de Windows**

### Étape 4 : Installer Bundler et Jekyll
{: .no_toc }

1. Dans le terminal, exécutez `gem install bundler`.
2. Puis, dans le dossier de votre projet, exécutez `bundle install`.

**Étape 5 : similaires à celles de Windows**
{: .no_toc }

## Sur macOS (non testé)

### Installer Ruby
{: .no_toc }

1. macOS vient avec Ruby installé, mais vous pouvez installer une version plus récente via Homebrew en exécutant `brew install ruby`.
2. Ajoutez le chemin du Ruby de Homebrew à votre PATH en suivant les instructions affichées après l'installation.

**Étapes 1,3,4 et 5 : similaires à celles de Windows**

---
