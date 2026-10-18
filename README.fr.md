# Lab NAT : masquerade et redirection de port avec nftables dans une VM Ubuntu

[English](README.md) | [Français](README.fr.md) | [日本語](README.ja.md) | [Español](README.es.md)

[![check](https://github.com/mitsurutech/lab-nat-gateway/actions/workflows/check.yml/badge.svg)](https://github.com/mitsurutech/lab-nat-gateway/actions/workflows/check.yml)

Un **lab NAT** prêt à l’emploi pour la classe, issu de la [plateforme d’apprentissage MitsuruTech](https://mitsurutech.co.uk/fr/learning), où les établissements donnent à chaque élève ses propres machines et un terminal dans le navigateur. Chaque élève reçoit une VM Ubuntu dans laquelle un petit réseau est construit : un hôte de LAN privé, une **passerelle** et un serveur extérieur. Il configure sur la passerelle le **masquerade (NAT source)** et une **redirection de port (NAT de destination)** avec **nftables**, enregistre les règles pour qu’elles survivent à un redémarrage, et voit de ses propres yeux ce que voit l’autre extrémité. Rien n’est installé sur l’ordinateur de l’élève.

Une VM est une seule machine : chacun des trois éléments est donc un **espace de noms réseau** (network namespace) et chaque liaison une paire de câbles virtuels. Les vraies commandes fonctionnent, et c’est tout l’intérêt : `nft`, `conntrack -L`, `tcpdump`, `curl`, `ip route`.

## Le lab

| Lab | Fichier | Ce que fait l’élève | Page d’aide |
| --- | --- | --- | --- |
| Passerelle NAT | [`nat-gateway.lab.yaml`](nat-gateway.lab.yaml) | Trouver pourquoi un hôte privé ne reçoit aucune réponse de l’extérieur, traduire ses adresses sur la passerelle, rediriger un port de l’extérieur vers une page web du LAN, autoriser ce flux et rien de plus large, puis enregistrer le jeu de règles | [Lab passerelle NAT](https://mitsurutech.co.uk/fr/help/lab-nat-gateway) |

Le réseau : un hôte de LAN sur `10.0.0.0/24`, la passerelle entre lui et un réseau extérieur, `203.0.113.0/24` (une plage réservée à la documentation), et un serveur extérieur qui répond à chaque requête avec l’adresse d’où il l’a vue arriver. L’extérieur n’a aucune route de retour vers le LAN, comme sur Internet. Le lab dépose dans le dossier personnel de l’élève un README qui dit quoi faire et ne donne aucune des réponses.

Il comporte une **vérification** (check), un script qui note une VM rendue sans que l’enseignant ait à la lire. Elle s’exécute sur une copie de la machine enregistrée après un démarrage à neuf, de sorte qu’elle ne voit que ce que l’élève a enregistré dans `/etc/nftables.conf`, et **elle note ce que fait le réseau, pas la façon dont les règles sont écrites** : une requête de l’hôte du LAN doit atteindre le serveur extérieur, qui doit avoir vu l’adresse extérieure de la passerelle ; une requête de l’extérieur vers le port redirigé doit atteindre la page du LAN, qui doit avoir vu l’adresse du client lui-même (un proxy sur la passerelle ne passe donc pas) ; l’autre page du LAN et tous les autres ports doivent rester fermés ; et la chaîne forward de la passerelle doit toujours bloquer ce qu’on ne lui a pas dit d’autoriser.

## Ce que contient le fichier

Le fichier `.lab.yaml` est une **définition de lab VM** complète, exactement ce que la plateforme exécute :

| Clé | Ce que c’est |
| --- | --- |
| `cpus`, `memory`, `disk` | la taille de la VM de chaque élève : 1 CPU, 1024 Mo et 10 Go |
| `open` | les ports qui s’ouvrent dans le navigateur : aucun, le lab est le terminal |
| `setup` | un script exécuté une fois, en root, au premier démarrage de la VM de chaque élève : il installe `nftables`, `conntrack`, `curl` et quelques outils depuis l’archive d’Ubuntu en une dizaine de secondes, et écrit la commande `netlab`, la topologie, le `/etc/nftables.conf` de départ et le README du lab |
| `check` | le script de notation, exécuté en root sur une copie du disque enregistré après un démarrage à neuf ; son code de sortie est le résultat et `0` signifie réussite |

La commande `netlab` qu’il installe : `netlab status` liste les machines, `netlab shell gw` ouvre un shell dans l’une d’elles, et `netlab reset` remet les règles de départ et relance le lab.

## Utilisation sur la plateforme

**Pour le donner comme travail à une classe :**

1. Dans votre module, ajoutez une tâche et choisissez la carte **Lab VM Ubuntu**.
2. Appuyez sur l’exemple Passerelle NAT, ou collez le contenu du fichier ci-dessus. Utiliser un exemple remplace le texte, et Annuler rétablit le vôtre.
3. Publiez et essayez d’abord votre propre copie. Chaque élève reçoit sa propre VM, conservée d’une session à l’autre.
4. Distribuez-la comme n’importe quelle tâche. Une fois rendue, lancez la vérification sur la ligne d’un élève dans la liste de la classe : elle démarre une copie de sa machine enregistrée, sans la réenregistrer, et indique si elle a réussi et ce qu’elle a affiché.

Le [guide des labs VM Ubuntu](https://mitsurutech.co.uk/fr/help/labs) du manuel et sa [page des labs d’exemple](https://mitsurutech.co.uk/fr/help/sample-labs) disent le reste.

Pas encore de compte pour votre établissement ? [Votre première année est gratuite](https://mitsurutech.co.uk/fr/pricing) jusqu’à 50 élèves, et [la création de compte](https://mitsurutech.co.uk/fr/signup) ne demande qu’une adresse e-mail.

## Vérification du fichier

À chaque push, `.github/workflows/check.yml` vérifie que le fichier est un YAML valide comportant les cinq clés ci-dessus et que `setup` et `check` tiennent chacun dans la limite de 16 Ko de la plateforme. Il ne démarre pas de VM : cela se fait sur les machines de la plateforme.

## Ce qui ne fonctionne pas dans un lab

La VM atteint l’archive de paquets d’Ubuntu, PyPI, npm et GitHub, et rien d’autre : un lab ne peut donc pas être dirigé vers un hôte sur Internet. C’est une seule VM, donc l’« extérieur » est un espace de noms à l’intérieur et non le véritable Internet.

## À propos

Réalisé par [Mitsuru Technologies](https://mitsurutech.co.uk/fr) pour la [plateforme d’apprentissage MitsuruTech](https://mitsurutech.co.uk/fr/learning) : Python, Java, Go, Node, SQL et Linux dans le navigateur pour les écoles et les établissements, avec des labs de conteneurs, des labs VM, des applications web déployées depuis GitHub et des quiz qui se corrigent tout seuls. Essayez du code sans compte dans le [bac à sable](https://mitsurutech.co.uk/fr/playground).

D’autres labs du même éditeur : [lab-netlab](https://github.com/mitsurutech/lab-netlab) (routage statique, OSPF et BGP), [lab-juice-shop](https://github.com/mitsurutech/lab-juice-shop), [lab-dvwa](https://github.com/mitsurutech/lab-dvwa), [lab-webgoat](https://github.com/mitsurutech/lab-webgoat), [lab-mysql-phpmyadmin](https://github.com/mitsurutech/lab-mysql-phpmyadmin), [lab-git-gitea](https://github.com/mitsurutech/lab-git-gitea) et [lab-sample](https://github.com/mitsurutech/lab-sample).

## Licence

Cette œuvre est sous licence [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/). Vous pouvez la partager et l’adapter pour apprendre et enseigner, à condition de citer Mitsuru Technologies Ltd, de ne pas l’utiliser commercialement et de partager tout ce que vous en tirez sous la même licence. Le texte complet est dans le fichier `LICENSE`.

nftables et conntrack-tools sont sous licence GPL-2.0 et sont installés depuis l’archive d’Ubuntu au démarrage d’une VM ; rien de leur code n’est distribué ici.
