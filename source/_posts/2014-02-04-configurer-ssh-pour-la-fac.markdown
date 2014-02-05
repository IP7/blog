---
layout: post
title: "Configurer SSH pour la fac"
date: 2014-02-04 22:36:16 +0100
author: Baptiste Fontaine
comments: true
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

C’est long, et avec en prime le fait de devoir la phrase de passe de la clef à
chaque fois. On peut faire mieux.

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
IP à la place d’un nom de domaine. Vous pouvez maintenant utiliser un de vos
alias à la place du nom du serveur.

Pensez à utiliser un alias à chaque fois que vous devez utiliser régulièrement
un long nom de domaine ou une IP fixe impossible à retenir. Notez aussi que
cette astuce marche pour `scp`, vous pouvez maintenant faire la commande
suivante pour copier un fichier de `nivose` en local :

    scp ni:mon_fichier.c .

Cette commande copie le fichier `~/mon_fichier.c` depuis `nivose` dans le
répertoire courant.

## Éviter de taper son mot de passe

Il ne suffit que d’une ligne de commande pour supprimer la demande de mot de
passe pour chaque connexion à un serveur. Exécutez la commande suivante depuis
votre machine locale :

    ssh-copy-id -i ~/.ssh/id_rsa.pub nivose

Exécutez la même commande pour chaque serveur. Votre mot de passe vous sera
demandé, mais essayez ensuite de vous connecter avec `ssh`, et vous serez
connecté directement sans avoir entré de mot de passe.

Si vous n’avez pas `ssh-copy-id`, utilisez la commande suivante:

    cat ~/.ssh/id_rsa.pub | ssh nivose 'cat >> ~/.ssh/authorized_keys'

Cette ligne fait la même chose que `ssh-copy-id`, elle concatène votre clef
publique dans le fichier `~/.ssh/authorized_keys` sur `nivose`. On notera la
syntaxe au passage qui permet d’exécuter une commande sur le serveur sans
ouvrir de session au préalable, par exemple :

    ssh nivose ls

Cette commande exécute `ls` sur `nivose` via SSH. C’est plus court que de se
connecter, exécuter `ls` et se déconnecter.

## Tunnels SSH

Il arrive parfois que le serveur qui permet d’accéder aux emplois du temps et
aux notes en ligne ne soit pas accessible de l’extérieur. C’est le cas
actuellement (4 février 2014). La solution est de passer via `lucien` pour être
ainsi *dans* dans la fac et accéder à la page Web. On peut utiliser un
navigateur textuel dans la session SSH, utiliser l’option `-X` de `ssh` puis
lancer Firefox ou équivalent mais il y a plus simple : les tunnels SSH.

La syntaxe est la suivante :

    ssh -N <hostname SSH> -L <port local>:<hostname cible>:<port cible>

Ici, on souhaite accéder à `magma.informatique.univ-paris-diderot.fr`, port
`2201`, en passant par `lucien`. En supposant que vous avez les mêmes alias que
moi, ça donne ça :

    ssh -N lu -L 2201:magma.informatique.univ-paris-diderot.fr:2201

Il suffit maintenant d’ouvrir votre navigateur à l’adresse `localhost:2201`. Le
navigateur se connecte donc en local au port `2201` (le premier `2201` de la
commande ci-dessus, vous pouvez changer le numéro de port si vous préférez), et
via un tunnel SSH via `lucien` (dont un des alias donné en exemple plus haut
est `lu`) se connecte à `magma.…` sur le port `2201` (le second `2201` de la
commande). Vous pouvez maintenant accéder à votre emploi du temps et vos notes
comme si vous étiez sur une machine de l’UFR. Remplacez simplement `magma.….fr`
dans toutes les URL par `localhost`.

Une fois votre consultation terminée vous pouvez fermer le tunnel avec un bon
vieux `^C`.
