# Compte rendu SAE3D04

<link rel="stylesheet" type="text/css" href="style.css">

</br></br>

## Mettre en place une infrastructure virtualisée

</br></br>

> Ce projet vise à comparer différents systèmes de virtualisation pour remplacer l'environnement VMWare actuellement en place. L'expertise a été réalisée dans un contexte professionnel et a impliqué l'installation et la configuration de serveurs Windows et Proxmox. Les progrès de l'installation ont été documentés pour créer une documentation utile et explicative de nos actions. Le compte rendu général reprend les comptes rendus de chaque partie.

</br></br>

## Table des matières

</br>

1. Gestion de projet
2. Proxmox
3. Hyper-V et Windows Server
4. VPN
5. Comparatif et conclusion

</br></br></br>

Avec la participation de Julien Alleaume, Ilker Onay, Mathieu Puig et Ndeye Codou Touré

## Gestion de projet

### Le projet

Le projet est basé sur une situation professionnelle, qui consiste en une expertise ayant pour but de comparer différents systèmes de virtualisation afin de remplacer l'environnement VMWare.

#### Nos objectifs

Pour réaliser ce projet, notre objectif est de comparer VMware, Hyper-V (la solution de virtualisation de Windows) et Proxmox (une solution de virtualisation open source gratuite).

Pour ce faire, nous avons décidé d'installer ces deux solutions directement sur des serveurs physiques. Nous nous fixons donc comme objectif d'installer deux serveurs Hyper-V et trois serveurs Proxmox afin de faire fonctionner CEPH.

Pour résumer :

* 2 serveurs Windows qui communiquent entre eux pour réaliser des migrations à chaud
* 3 serveurs Proxmox avec un système de partage de fichiers CEPH pour effectuer les migrations à chaud.

#### Notre organisation

Nous avons d'abord noté dans un référentiel git le sujet qui est accessible à tous afin que tous les membres du groupe comprennent bien nos objectifs.

![Sujet](./img/Capture%20d’écran%20du%202022-12-14%2012-08-19.png)

Après cela, pour simplifier la gestion des tâches et avoir une organisation claire du travail à exécuter, nous avons pris l'initiative de réaliser un projet "SAE Cloud" sur Jira Work Management afin de visualiser les tâches de chacun et de suivre l'avancement du projet.

![jira](./img/Capture%20d’écran%20du%202022-12-14%2014-21-27.png)

En plus pour une meilleure gestion des serveurs, nous avons mis en place des crédits en ligne accessibles par tous et modifiables selon nos configurations.

![idrac](./img/idrac.png)

![servP](./img/sysvirt.png)

![servW](./img/sysvirtwin.png)

Parmi les informations utiles incluses dans ces crédits, nous avons :

![infoUtile](./img/info.png)

![vpn](./img/vpn.png)

#### Taches realiser au cours du projet

Lors de ce projet, nous avons réalisé plusieurs tâches réparties sur deux semaines. Elles sont résumées dans le schéma suivant :

![time-line](./img/Capture%20d’écran%20du%202022-12-14%2014-33-49.png)

Pour résumer, nous avons pu réaliser toutes les installations que nous voulions faire sur Proxmox. Nous avons également pu installer un VPN qui nous a permis d'administrer nos serveurs à distance. Du côté de Windows, nous avons pu découvrir l'environnement Windows Server et Hyper-V, et nous somme allez jusqu'a la migration a chaud.

#### Conclusion et ameloiration possible

On peut donc dire que malgré l'ampleur de notre projet, grâce au travail fourni par les membres du groupe, nous avons atteint nos objectifs principaux.

Ce projet nous a montré que la coordination du travail est très importante, car nous avons pu voir qu'un manque de coordination pouvait mener à des désaccords au sein d'un groupe de travail. En vue d'améliorer notre travail en groupe à l'avenir, nous pourrions mettre en place des méthodes de gestion de projet plus efficaces pour éviter ces désaccords et atteindre nos objectifs de manière plus efficace.

## Proxmox

> Vous pouvez retrouvez des commandes à la fin du documents

### 1.Installation

#### A quoi sa sert ?

Ici nous allons définir les différents termes qu'on va utiliser par la suite et leur fonctionnement.

CEPH : C'est une solution libre de stockage distribué qu'on peut retrouver sur proxmox.

Monitor:Les moniteurs (Mons) Chaque cluster Ceph nécessite la mise en œuvre de moniteurs installés sur des serveurs indépendants. Ces moniteurs sont utilisés par les clients Ceph pour obtenir la carte la plus à jour du cluster. Les moniteurs s’appuient sur une version modifiée du protocole Paxos pour établir entre eux un consensus sur la cartographie du cluster

Manager:

OSD: À chaque OSD correspond un démon chargé de stocker les données, de les répliquer ou de les redistribuer en cas de défaillance d’un équipement. Chaque démon OSD fournit aussi des informations de monitoring et de santé aux moniteurs Ceph. Un cluster Ceph doit à minima disposer de deux démons OSD (3 sont recommandés) pour démarrer.

Pool: Un cluster Ceph stocke les données sous forme d’objets stockés dans des partitions logiques baptisées “pools”. À chaque pool Ceph correspond un ensemble de propriétés définissant les règles de réplications ou le nombre de groupes de placement dans le pool.Par exemple, si l’on a spécifié trois copies et que le cluster dispose de trois nœuds, la triple réplication permet de survivre à deux pannes de nœuds ou à la panne de deux disques.

#### Prérequis

Pour l'installation CEPH il vous au minimum 2 partitions de disques virtuels par serveur. Nous sommes sur des serveurs qui ont déjà était utiliser donc dans notre cas il faut `supprimer` les partitions de disques virtuals et par la suite on obtient une seul partitions qu'on `clear`. Car lors de la créations des OSD Ceph il faut allouer une partitions de disques si on déjà étaient allouer a d'ancien noeud il se peut que proxmox vous bloquer à cette étape et vous devez tout recommencer, c'est pour celà qu'il vaut mieux s'en assurer dès le débuts.

C'est aussi une occasion d'activer `NPar+ SR-IOV`, SR-IOV crée 8 carte réseaux virtuelle et NPar permet de gérer la bande passante sur celle-ci (exemple : Si on a juste une carte réseaux occupée toute la bande passante lui appartient mes si ont a deux cartes réseaux 'occuper' la bande passante est diviser en 2 , etc,etc,...)

L'activation de NPAR+ SR-IOV :
Chemin : `Device Settings -> Votre carte 10G -> Device Level Configuration -> Virtualization Mode : NPar+ SR-IOV`

![img](./img/screen_prerequis/1.png)

> On peut observer qu'ils ont bien était activées.

![img](./img/screen_prerequis\3.png)

Suppression des cartes réseaux :
Chemin : `Device Settings -> Intgrated RAID Controller
-> Virtual Disk Management`

![image]( "img\screen_prerequis\2.png")

#### 1.1 Promox

##### 1.1.1 BIOS

Dans un premier temps il faut booter son iso sur le IDRAAC, pour ceci il faut cliquer sur `Connecter le média virtuel`, ajouter l'iso Proxmox puis le Mapper

<img src="img\Screen\Capture du 2022-12-07 10-31-11.png">

> Vous pouvez cliquer sur `fermer`

Choisir de le booter sur `Virtual CD / DVD / ISO`

<img src="img\Screen\Capture du 2022-12-07 10-31-53.png">

`Réinitialiser le système (démarrage à chaud)`

<img src="img\Screen\Capture du 2022-12-07 10-32-27.png">

##### 1.1.2 Proxmox

Appuyer `Entrer`

<img src="img\Screen\Capture du 2022-12-07 10-39-31.png">

Cliquer sur `I agree`

<img src="img\Screen\Capture du 2022-12-07 10-41-24.png">

Il faut choisir la partition de disque, puis cliquer sur `Next` ( sinon on peut cliquer sur `options` pour modifier )

<img src="img\Screen\Capture du 2022-12-07 10-41-57.png">

Selectioner votre Pays/Zone, puis cliquer sur `Next`

<img src="img\Screen\Capture du 2022-12-07 10-42-15.png">

Mot de passe : rftgy#123 (dans notre cas), saissir une email valide

<img src="img\Screen\Capture du 2022-12-07 10-43-58.png">

> IP address : Adresse que récupéra votre serveur Proxmox

Gateway : Dans mon cas c'est la passerelle par défauts de la salle

DNS : Dans mon cas celui de l'IUT

<img src="img\Screen\Capture du 2022-12-07 10-48-27.png">

Cliquer `Install`

<img src="img\Screen\Capture du 2022-12-07 10-48-45.png">

<img src="img\Screen\Capture du 2022-12-07 10-51-55.png">

##### Connection

###### Connection via console

Exemple :

```bash
Login : root
Mot de passe : rftgy#123
```

<img src="img\Screen\Capture du 2022-12-07 11-29-04.png">

<img src="img\Screen\Capture du 2022-12-07 11-29-49.png">

###### Connection graphique

<img src= "img\Screen\Capture du 2022-12-07 11-30-22.png">

<img src= "img\Screen\Capture du 2022-12-07 11-31-41.png">

#### 1.2 CEPH

##### 1.2.1.Cluster

Pour l'installation de Ceph il faut crée un cluster avec minimum 3 noeud est recommandée.

Donc pour celà, sur la machine "hôte" il faut crée un cluster puis partager son code au autres noeuds.

Pour ce faire aller dans `Cluster` puis dans `Create a cluster` est il s'affichera la page ci-dessous.

<img src= "img\Screen_ceph\Capture du 2022-12-12 15-49-42.png">

Une fois dedans Entrée un nom de cluster par exemple `CephIlker` car ce Cluster va me servir pour faire du CEPH

Quand on clique sur le Cluster on a maintenant accès au boutton `Cluster Join Information` celui-ci va permettre a vos autres noeuds de facilement rejoindre le cluster (la meilleur façon de copier et de cliquer sur `Copy Information` pour être sur de ne pas oublier le moindre caractères à copier)

<img src="img\Screen_ceph\Capture du 2022-12-12 15-50-10.png">

Après celà, il nous reste plus qu'à aller dans la partie cluster sur les autres noeuds et cliquer sur `Join Cluster` pour rejoindre le cluster a fin de faire les liens entre eux. Dans la catégorie Information il faut copier le "code", puis dans password entrer le mot de passe du serveur qui détient le cluster dans notre cas tout le mot de passe proxmox est `rftgy#123`. Il nous reste plus cas choisir le cluster network qu'il vous propose.

<img src="img\Screen_ceph\Capture du 2022-12-12 15-51-16.png">

##### 1.2.2.Ceph

##### Etape 1

Après avoir crée est rejoint notre cluster avec les deux autres serveurs il faut installer notre CEPH.

Pour celà il faut cliquer sur CEPH dans la catégorie de notre noeud numéro un, deux et trois. Puis cliquer sur install ( Attention en aucun cas ne fermer pas la page car votre installation risque de crash et vous devez tout recommencer). Une fois avoir cliquer sur l'install il se fera automatiquement seulement dans le premier noeud vous aurez a choisir votre `Public Network` qui est l'IP de votre serveur numéro un, pour les autres l'installation ce fait automatiquement.

<img src= "img\Screen_ceph\Capture du 2022-12-12 16-08-54.png">

##### Etape 2

Sur votre noeud dans la catégorie `CEPH` puis `Monitor` ajouter les moniteur et manager deux et trois en cliquant sur `Create` dans la partie Monitor ou Manager

<img src="img\Screen_ceph\Capture du 2022-12-12 16-30-27.png">

<img src="img\Screen_ceph\Capture du 2022-12-12 16-30-16.png">

##### Etape 3

Sur chaque noeud il va falloir crée une OSD, donc depuis la catégorie `CEPH` puis `OSD` cliquer sur crée est sélectionnées votre partition de disque libre

<img src= "img\Screen_ceph\Capture du 2022-12-13 15-47-25.png">

Une fois fini :
<img src= "img\Screen_ceph\Capture du 2022-12-13 15-50-27.png">

##### Etape 4

Il faut maintenant crée une pool depuis le pve1, pour faire celà aller dans `CEPH` puis `Pools`, cliquer sur `Create`

<img src= "img\Screen_ceph\Capture du 2022-12-13 16-24-38.png">

##### Etape 5

Pour voir le bon fonctionnement de toutes votre installation il faut vérifier l'état de santé de notre CEPH.
Pour cela il suffit de cliquer sur `CEPH` :

<img src= "img\Screen_ceph\Capture du 2022-12-13 16-55-33.png">

### 2.Utilisation

#### 2.1 Création d'une VM

La création de VM est un peu particulier sur proxmox l'installation via ISO, ça nécessitent de retirer la carte réseaux et de le r'ajouter après l'installation mais aussi de retirer le CD/DVD après l'installation pour pouvoir migrer la VM.

Pour crée votre VM il faut d'abord upload votre iso, rendez-vous dans votre noeud et cliquer sur `Image ISO` et cliquer sur `UPLOAD`,puis `Select File` pour selectionner votre ficher, finir en cliquant sur `UPLOAD`

<img src="img\screen_create_vm\create_vm1.png">

Pour crée une VM, maintenant vous devez cliquer sur `Create VM` en haut à droite :

Il vous faut choisir le noeud d'appartenance, une ID est donner par défauts changer la si vous le souhaitez, donner un nom à votre VM.

<img src="img\screen_create_vm\create_vm2.png">

Choissisez l'image ISO que vous souhaitez que votre VM doit prendre.

<img src="img\screen_create_vm\create_vm3.png">

Je laisse par défauts

<img src="img\screen_create_vm\create_vm4.png">

Définisser si vous voulez utiliser le stockage local ou ceph depuis `Storage` et la taille de Disk dans mon cas 8Go est largement suffisants

<img src="img\screen_create_vm\create_vm5.png">

Quelque chose d'important sur proxmox est que pour chaque VM vous pouvez limiter la bande passante seulement sur la VM sans passer par une limitation au niveau du port comme VMWare

<img src="img\screen_create_vm\create_vm6.png">

On peut augmenter le nombre de coeurs allouer

<img src="img\screen_create_vm\create_vm7.png">

On peut augmenter la RAM (mémoire vice) allouer (Valeur en Mo)

<img src="img\screen_create_vm\create_vm8.png">

Ici on doit crée une vm sans carte réseaux pour l'ajouter après l'installation ISO fini, car dans l'établissement les installations de paquets et autres nous prend beaucoup de temps à cause de la connection internet. Donc pour éviter les installations de paquets ou autres il vaut mieux désactiver la création de carte réseaux (je vous montre par la suite comment la crée). Cliquer sur ``No network device`.

<img src="img\screen_create_vm\create_vm9.png">

Vous pouvez relire les informations de création de votre VM pour vous assurer de votre configuration souhaitez.

Si vous souhaitez le démarrage après la création cliquer sur `Start after created` (en bas à gauche)

<img src="img\screen_create_vm\create_vm10.png">

On peut observer qu'on a bien notre vm de crée mais il n'a pas de connection internet

<img src="img\screen_create_vm\create_vm11.png">

Cliquer sur votre VM,ensuite dans le menu déroulant cliquer sur `Hardware`, puis sur `Add` et selectionner `Network Device`. Selectionner votre bridge et appuyer sur `Add`

<img src="img\screen_create_vm\create_vm12.png">

On vérifie bien en ping 8.8.8.8, on a bien à une connection internet.
<img src="img\screen_create_vm\create_vm13.png">

#### 2.2 Migration d'une VM

Pour migrer une VM après l'installation via ISO il necéssitent de se rendre dans la partie `Hardware` de votre VM et de modifier le `CD/DVD Drive` est de le mettre en mode `Do not use any media`

<img src="img\screen_migrate\migrate1.png">

<img src="img\screen_migrate\migrate2.png">

 Migration à froid :

Ici nous avons notre VM nommée "CentosMini" dans le noeud "pve1".

<img src="img\screen_migrate\migrate3.png">

On clique droit dessus puis sur `Migrate` et on choissie vers quel noeud on voeud le migrer (mauvais screen j'ai migrer vers le noeud 3 on pourra le voir dans le screen suivants)

<img src="img\screen_migrate\migrate4.png">

On peut observer le message (to node 'pve3', donc vers le noeud 3), la migration à réussis

<img src="img\screen_migrate\migrate5.png">

<img src="img\screen_migrate\migrate6.png">

 Migration à chaud :

Maintenant, on va démarrer la VM pour faire une migration à chaud

<img src="img\screen_migrate\migrate7.png">

On migre au noeud de départ donc la une (pve1)

<img src="img\screen_migrate\migrate8.png">

On peut voir que la migration à encore réussis

<img src="img\screen_migrate\migrate9.png">

<img src="img\screen_migrate\migrate10.png">

#### 2.3 Création d'une template

A partir d'une VM nous pouvons crée une template en fessant clique droit `Convert to template`

<img src="img\screen_template\template1.png">

Pour déployer une VM depuis une template il faut faire clique droit dessus est choissir `Clone`, donner un nom a votre nouvelle VM et le mode de clonage par exemple Full Clone

<img src="img\screen_template\template2.png">

Le tour est jouer vous avez crée votre VM

<img src="img\screen_template\template3.png">

#### 2.4 Commande qui mon était utiles

Cette partie regroupe les commande que j'ai du utiliser lors de cette SAE pour proxmox

Pour supprimer un cluster :

```bash
#A taper sur le noeud d'appartence du cluster
systemctl stop pve-cluster
systemctl stop corosync
pmxcfs -l
rm /etc/pve/corosync.conf
rm /etc/corosync/* -rf
killall pmxcfs
rm /var/lib/corosync/* -f

systemctl start pve-cluster
```

Pour supprimer un noeud :

```bash
rm -r /etc/pve/[node_name]
```

Pour modifier l'IP du serveur proxmox :

```bash
vi /etc/network/interfaces
systemctl restart networking.service
```

## Hyper-V et Windows Server

## VPN Wireguard VPN pour administration a distance

Ici la salle est sur le resau 10.202.0.0/16 et le VPN sur le 172.20.0.0/16.
Le resau de l'iut et nater et il est imposible de faire du port-forwarding, il est donc imposible de metre directement notre serveur VPN sur le resau de l'iut.
Pour regler ce probleme nous allont metre le serveur VPN sur un VPS et le serveur ProxMox de l'iut sera un client qui auras un keepalive qui conserveras le tunelle vpn. Grace a cette architecture et la mise en place de routage et de regle de NAT des packet ont peut acceder a la salle cloud depuis le serveur proxmox avec sont IP.

Voici un schéma de notre installation.

![img](./img/vpn.svg)

### Configuration du serveur VPS par commande

Voici comment on peut configurer wireguard en ligne de commande. Ici ce sont les commande de configuration du serveur VPS.

```bash
wg genkey > priv 
```

Ici on génère la clé privée de notre client (dans ce cas celle du serveur).

```bash
sudo ip link add wg0 type wireguard 
```

Ici on crée notre nouvelle interface de réseau de type wireguard.

```bash
sudo ip a add 172.20.20.1/16 dev wg0 
```

Ici, on lui donne l'adresse IP que l'on souhaite. Dans ce cas, on utilise une adresse IP privée dans un réseau en 172.20.0.0/16 pour éviter tout conflit avec la salle cloud ou avec les réseaux locaux des clients.

```bash
wg set wg0 private-key ./privatekey 
```

Ici on attribue la clé que l'on a générée précédemment.

```bash
sudo ip link set wg0 up 
```

Ici, on active l'interface que nous avons créée.

```bash
sudo wg set wg0 peer xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx allowed-ips 172.20.20.2/32 
```

Après avoir suivi la procédure précédente pour les clients, on peut les ajouter en utilisant cette commande. Nous spécifions leur clé publique et leur adresse IP à laquelle nous autorisons l'accès. Ici, nous autorisons une adresse IP en /32 pour garantir qu'uniquement cette adresse IP puisse se connecter.

```bash
sudo wg set wg0 peer xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx allowed-ips 172.20.20.3/32,10.202.0.0/16 
```

De même ici, mais en ajoutant les adresses IP de la salle afin de pouvoir communiquer avec.

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward 

```

Ici, on active le routage de paquets pour que les pairs puissent envoyer des paquets au réseau 10.202.0.0/16.

```bash
ip route add 10.202.0.0/16 via 172.20.20.3 
```

On ajoute donc la route pour le résau.

#### Côté client

Du coté client on doit reprendre aproximativement les configurations du VPS sauf que l'on doit set le vps en tant que peer.

```bash
sudo wg set wg0 peer xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx allowed-ips 172.20.0.0/16,10.202.0.0/16 endpoint XX.XX.XX.XX:XX #Ici, on met toujours la clé publique et les IP autorisées, mais on rajoute également l'IP publique du serveur VPN.
```

### Sauvegarder les configurations que l'on a fait

Les configuration que l'on a fait en commande sont volatile, il faut donc les sauvegardes dans un fichier de configuraion pour les conserver.

```bash
wg showconf wg0 > /etc/wireguard/wg0.conf 
```

Ici, on affiche les configurations puis on les redirige vers un fichier de configuration.

```bash
systemctl enable --now wg-quick@wg0 
```

Ici, pour le VPS et le serveur de l'IUT, on active l'option qui permet à l'interface de se réactiver automatiquement lors du redémarrage.

### Configuration du VPS

Voici le fichier de configuration du VPS. Que l'on quelque peut modifier afin que l'on ait plus a taper de comande apres l'activation de l'interface. De plus avec les fichier de donfiguration, lors de l'activation de l'interface les routes sont ajouter automatiquement.

```bash
[Interface]
ListenPort = 52403 #Ici, on fixe le port pour qu'il ne change pas à chaque redémarrage.
Address = 172.20.20.1/16 #Adresse du VPS dans notre VPN
PrivateKey = xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx #Clé privée générée plutôt.
PostUp=echo 1 > /proc/sys/net/ipv4/ip_forward #Activation au démarrage du routage des paquets

[Peer]
PublicKey = xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx #Clé publique du peer
AllowedIPs = 172.20.20.2/32 #IP autorisée
Endpoint = 193.57.121.159:65526 #IP de l'endpoint (Généré automatiquement, si le peer se connecte depuis une autre IP, cela ne pose pas de problème)

[Peer]
PublicKey = xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
AllowedIPs = 172.20.20.3/32, 10.202.0.0/16
Endpoint = 194.199.227.10:35924
```

### Configuration du serveur IUT

Voici le fichier de configuration du serveur Proxmox coté iut. (Client VPS)

```bash
[Interface]
ListenPort = 35924
PrivateKey = xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
Address = 172.20.20.3/16
PostUp=echo 1 > /proc/sys/net/ipv4/ip_forward && iptables -t nat -A POSTROUTING -s 172.20.0.0/16 -o vmbr0 -j MASQUERADE #Ici, on autorise le routage de paquets et on ajoute une règle iptables pour NATer les paquets sur le réseau.
PostDown=iptables -t nat -D POSTROUTING -s 172.20.0.0/16 -o vmbr0 -j MASQUERADE #Ici on désactive le NAT.

[Peer]
PublicKey = xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
AllowedIPs = 172.20.0.0/16
Endpoint = XX.XX.XX.XX:XX #ip:port
PersistentKeepalive = 25 #Ici on met un keepalive de 25 secondes qui permet de maintenir le tunnel VPN en fonctionnement.
```

### Configuration du poste Client

Voici le fichier de configuration du poste client.

```bash
[Interface]
ListenPort = 58432
PrivateKey = xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
Address = 172.20.20.2/16

[Peer]
PublicKey = xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
AllowedIPs = 172.20.0.0/16, 10.202.0.0/16
Endpoint = XX.XX.XX.XX:XX #ip:port
```

### Activation

Pour activer l'interface au demarage :

```bash
sudo systemctl enable --now wg-quick@wg0
```

Pour l'activer ponctuellement :

![img](./img/Capture%20d’écran%20du%202022-12-11%2022-07-30.png)

### Setup IP et ping

Ici mon interface resau wlp1s0 est connecter a un partage de connection en 4G. Avec l'interface wg0 d'activer et de connerter je peux bien ping le resaux de l'IUT.

![img](./img/Capture%20d’écran%20du%202022-12-09%2018-29-05.png)

### TraceRoute

Depuis cette foi-ci le resau fibre de mon appartement je realise un trace route, je passe bien par mon VPS puis le serveur de l'IUT pour ensuite arriver sur la salle.

![img](./img/Capture%20d’écran%20du%202022-12-11%2022-07-20.png)

### Desactivation

Pour desactiver l'interface :

![img](./img/Capture%20d’écran%20du%202022-12-11%2022-07-41.png)

## Comparatif et conclusion

> Dans cette partie on vas raliser un comparatif des trois solution de virtualisation avec plusieur critaire, et nous rendront notre avis sur quel systeme nous semble le plus adequoit en remplacement de VMWare.

## Performances

> Il est important de comparer les performances des deux systèmes en termes de temps d'exécution des tâches et de consommation des ressources (mémoire, processeur, etc.).

En terme de perfomance pure les diferent systeme ont des performance asser similiaire, que ce soir sur de la virtualisation de machine linux ou windows les diferences de performance entre les diferent systemes sont asser negligable en tent que critaire de selection. Même si aprês plusieur comparaisont et sur de plus grande echelle on peut constater de meilleur performance pour windows serveur.

## Flexibilité et facilité d'utilisation

> Il est important de comparer la facilité d'utilisation des deux systèmes, notamment en ce qui concerne la création et la gestion des machines virtuelles. La compatibilité avec différents systèmes d'exploitation et applications est également un aspect à prendre en compte

## Coûts

> Il est important de comparer les coûts associés à chaque système de virtualisation, notamment en termes de licences et de support technique

En ce qui concerne les coûts, les deux systèmes sont assez différents : Proxmox est gratuit en théorie mais en production, il est absolument nécessaire d'avoir un support qui est donc payant.

En ce qui concerne Windows Server, il fonctionne sous forme de licence (5 ans de durée de vie) et nécessite plusieurs licences différentes : une pour le serveur maître et d'autres pour les serveurs esclaves. Les licences en question ont une durée de vie de 5 ans.

Comparer les prix est assez difficile vu les différences de fonctionnement de chaque système, mais pour résumer, Windows Server sera plus rentable sur une infrastructure plus grande.

Par exemple, si on prend 11 serveurs (1 maître, 10 esclaves) sur 5 ans, on aura :

Pour Windows, ce sera une licence Datacenter à 6200 $ et dix licences Standard à 1070 $, ce qui reviendra à environ 17 000 $ pour les 5 ans complets.

Tandis que pour Proxmox, l'assistance coûte environ 950 $ (890 €) par an par CPU, donc dans ce cas, sur 5 ans, cela reviendra à environ 52 250 $.

En contrepartie, si on utilise une infrastructure plus petite, il sera plus rentable d'utiliser Proxmox.

## La sécurité

> Il est important de comparer les niveaux de sécurité des deux systèmes, notamment en termes de protection des données et de contrôle d'accès aux machines virtuelles

## La compatibilité matérielle

> Il est important de vérifier que les deux systèmes de virtualisation sont compatibles avec les différents composants matériels de votre ordinateur (carte graphique, disques durs, etc.)

## Les fonctionnalités avancées

> Il est utile de comparer les fonctionnalités avancées proposées par les deux systèmes, telles que la prise en charge de plusieurs systèmes d'exploitation en simultané, la gestion de la mémoire et du processeur en temps réel, etc

