---
layout: post
title: "Configurer SSH pour la fac"
date: 2014-02-04 19:36:16 +0100
author: Baptiste Fontaine
comments: true
published: false
categories:
- L3
- M1
- M2
- UFR
- Ligne de commande
---

Cet article est principalement destiné aux étudiants à partir de la L3,
puisqu’ils dépendent de l’UFR d’Informatique qui permet un accès SSH à la fac
qui n’est pas possible en L1 et L2. On suppose que vous savez déjà utiliser SSH
et donc que vous avez déjà créé un couple de clefs (comme expliqué
[ici][ssh-gen]). Les astuces présentées dans cet article sont bien entendu
applicables en dehors de l’UFR, pour n’importe quel serveur accessible en SSH.

[ssh-gen]: http://www.informatique.univ-paris-diderot.fr/wiki/doku.php?id=wiki:howto_connect#generation_des_cles

Deux serveurs sont accessibles de l’extérieur pour accéder à son compte à l’UFR
: `nivose` et `lucien`. Avec le sous-domaine de l’UFR on arrive donc à la
commande suivante :

    ssh dupont@nivose.informatique.univ-paris-diderot.fr

Avec en prime le fait de devoir la phrase de passe de la clef à chaque fois. On
peut faire mieux.

<!-- more -->

## Utiliser des alias

L’utilisation des alias permet de réduire la commande précédente à celle-ci :

    ssh nivose

Pour ce faire, il suffit d’éditer votre fichier `~/.ssh/config` (créez-le s’il
n’existe pas). Pour `nivose`, ajoutez les lignes suivantes :

```
Host nivose ni
    Hostname nivose.informatique.univ-paris-diderot.fr
    User dupont
    Protocol 2
```

La première ligne après `Host` indique le(s) alias que vous voulez utiliser.
Ici, j’utilise `nivose` et `ni` pour être encore plus court. Vous pouvez les
modifier et/ou en ajouter d’autres. En particulier, l’alias n’a pas de lien
avec le champ `Hostname` situé en dessous, vous pouvez tout-à-fait utiliser un
alias `fac` ou `toto` si vous voulez. La seconde ligne indique le domaine
complet du serveur. La ligne suivante indique le nom d’utilisateur à utiliser,
c’est la partie que vous utilisez devant `@` quand vous vous connectez, pour
l’UFR c’est votre nom en minuscules. La dernière ligne indique la version du
protocole SSH à utiliser, en pratique ça sera toujours la version 2, plus
sécurisée que la première version.

Vous pouvez ajouter d’autres champs en fonction du serveur, par exemple le
champ `Port` pour spécifier le port à utiliser s’il est différent du port
classic (22).

Pour ajouter d’autres serveurs (comme `lucien`), utilisez la même syntaxe dans
le fichier :

```
Host nivose ni
    Hostname nivose.informatique.univ-paris-diderot.fr
    User dupont
    Protocol 2

Host lucien lu
    Hostname lucien.informatique.univ-paris-diderot.fr
    User dupont
    Protocol 2
```

Et ainsi de suite. Le champ `Hostname` peut bien entendu contenir une adresse
IP à la place d’un nom de domaine.

## Éviter de taper son mot de passe

Il ne suffit que d’une ligne de commande pour supprimer la demande de mot de
passe pour chaque connexion à un serveur. Exécutez la commande suivante depuis
votre machine locale :

    ssh-copy-id -i ~/.ssh/id_rsa.pub nivose

TODO

