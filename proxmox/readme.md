# <div align="center"><ins> Installer et utiliser :</ins>  <br>Proxmox et CEPH</div>
>Vous pouvez retrouvez des commandes à la fin du documents
### <ins>1.Installation :</ins>
### <ins>A quoi sa sert ? </ins>
Ici nous allons définir les différents termes qu'on va utiliser par la suite et leur fonctionnement.<br>
CEPH : C'est une solution libre de stockage distribué qu'on peut retrouver sur proxmox.<br>
Monitor:Les moniteurs (Mons) Chaque cluster Ceph nécessite la mise en œuvre de moniteurs installés sur des serveurs indépendants. Ces moniteurs sont utilisés par les clients Ceph pour obtenir la carte la plus à jour du cluster. Les moniteurs s’appuient sur une version modifiée du protocole Paxos pour établir entre eux un consensus sur la cartographie du cluster<br>
Manager:<br>
OSD: À chaque OSD correspond un démon chargé de stocker les données, de les répliquer ou de les redistribuer en cas de défaillance d’un équipement. Chaque démon OSD fournit aussi des informations de monitoring et de santé aux moniteurs Ceph. Un cluster Ceph doit à minima disposer de deux démons OSD (3 sont recommandés) pour démarrer.<br>
Pool: Un cluster Ceph stocke les données sous forme d’objets stockés dans des partitions logiques baptisées “pools”. À chaque pool Ceph correspond un ensemble de propriétés définissant les règles de réplications  ou le nombre de groupes de placement dans le pool.Par exemple, si l’on a spécifié trois copies et que le cluster dispose de trois nœuds, la triple réplication permet de survivre à deux pannes de nœuds ou à la panne de deux disques<br>

### <ins>Prérequis : </ins>

Pour l'installation CEPH il vous au minimum 2 partitions de disques virtuels par serveur. Nous sommes sur des serveurs qui ont déjà était utiliser donc dans notre cas il faut `supprimer` les partitions de disques virtuals et par la suite on obtient une seul partitions qu'on `clear`. Car lors de la créations des OSD Ceph il faut allouer une partitions de disques si on déjà étaient allouer a d'ancien noeud il se peut que proxmox vous bloquer à cette étape et vous devez tout recommencer, c'est pour celà qu'il vaut mieux s'en assurer dès le débuts.

C'est aussi une occasion d'activer `NPar+ SR-IOV`, SR-IOV crée 8 carte réseaux virtuelle et NPar permet de gérer la bande passante sur celle-ci (exemple : Si on a juste une carte réseaux occupée toute la bande passante lui appartient mes si ont a deux cartes réseaux 'occuper' la bande passante est diviser en 2 , etc,etc,...)


<ins>L'activation de NPAR+ SR-IOV :</ins><br>
Chemin : `Device Settings -> Votre carte 10G -> Device Level Configuration -> Virtualization Mode : NPar+ SR-IOV` 
<br>
<img src= "img\screen_prerequis\1.png"><br>
>On peut observer qu'ils ont bien était activées.

<img src= "img\screen_prerequis\3.png">
<br><br>

<ins>Suppression des cartes réseaux :<br> 
Chemin : `Device Settings -> Intgrated RAID Controller
-> Virtual Disk Management`

<img src= "img\screen_prerequis\2.png">


<br><br><br>

### <ins>1.1 Promox  :</ins>
#### <ins> 1.1.1 BIOS</ins><br>
Dans un premier temps il faut booter son iso sur le IDRAAC, pour ceci il faut cliquer sur `Connecter le média virtuel`, ajouter l'iso Proxmox puis le Mapper
<br><br>
<img src ="img\Screen\Capture du 2022-12-07 10-31-11.png">
> Vous pouvez cliquer sur `fermer`
<br>


<br>Choisir de le booter sur `Virtual CD / DVD / ISO`<br>

<img src ="img\Screen\Capture du 2022-12-07 10-31-53.png"><br>

`Réinitialiser le système (démarrage à chaud)`<br>

<img src ="img\Screen\Capture du 2022-12-07 10-32-27.png">
<br><br>

---

#### <ins> 1.1.2 Proxmox : </ins>
<br>

Appuyer `Entrer`

<img src ="img\Screen\Capture du 2022-12-07 10-39-31.png">
<br><br>

Cliquer sur `I agree`<br>

<img src ="img\Screen\Capture du 2022-12-07 10-41-24.png">
<br><br>

Il faut choisir la partition de disque, puis cliquer sur `Next` ( sinon on peut cliquer sur `options` pour modifier ) <br>

<img src ="img\Screen\Capture du 2022-12-07 10-41-57.png">
<br><br>

Selectioner votre Pays/Zone, puis cliquer sur `Next `<br>

<img src ="img\Screen\Capture du 2022-12-07 10-42-15.png">
<br><br>

Mot de passe : rftgy#123 (dans notre cas), saissir une email valide <br>
<img src ="img\Screen\Capture du 2022-12-07 10-43-58.png">
<br><br>

>IP address : Adresse que récupéra votre serveur Proxmox
<br>Gateway : Dans mon cas c'est la passerelle par défauts de la salle<br>
DNS : Dans mon cas celui de l'IUT<br>

<img src ="img\Screen\Capture du 2022-12-07 10-48-27.png">
<br><br>

Cliquer `Install`<br>

<img src ="img\Screen\Capture du 2022-12-07 10-48-45.png">
<br><br>

<img src ="img\Screen\Capture du 2022-12-07 10-51-55.png">
<br><br> 

#### <ins>Connection :</ins>

##### <ins>Connection via console :</ins>

<br><ins>Exemple :</ins>
<br><br>Connecter vous :<br>
Login : root <br>
Mot de passe : rftgy#123<br>
<img src ="img\Screen\Capture du 2022-12-07 11-29-04.png">
<img src ="img\Screen\Capture du 2022-12-07 11-29-49.png">


##### <ins>Connection graphique :</ins>
<img src= "img\Screen\Capture du 2022-12-07 11-30-22.png">
<img src= "img\Screen\Capture du 2022-12-07 11-31-41.png">
<br><br><br><br>

---
<br><br>

### <ins>1.2 CEPH  :</ins>

#### <ins> 1.2.1.Cluster :</ins>
<br>


Pour l'installation de Ceph il faut crée un cluster avec minimum 3 noeud est recommandée.<br>
Donc pour celà, sur la machine "hôte" il faut crée un cluster puis partager son code au autres noeuds.

Pour ce faire aller dans `Cluster` puis dans `Create a cluster` est il s'affichera la page ci-dessous.
<br><br>
<img src= "img\Screen_ceph\Capture du 2022-12-12 15-49-42.png">
<br><br>
Une fois dedans Entrée un nom de cluster par exemple `CephIlker` car ce Cluster va me servir pour faire du CEPH
<br><br>

Quand on clique sur le Cluster on a maintenant accès au boutton `Cluster Join Information` celui-ci va permettre a vos autres noeuds de facilement rejoindre le cluster (la meilleur façon de copier et de cliquer sur `Copy Information` pour être sur de ne pas oublier le moindre caractères à copier)
<br><br>
<img src="img\Screen_ceph\Capture du 2022-12-12 15-50-10.png">

<br><br>
Après celà, il nous reste plus qu'à aller dans la partie cluster sur les autres noeuds et cliquer sur `Join Cluster` pour rejoindre le cluster a fin de faire les liens entre eux. Dans la catégorie Information il faut copier le "code", puis dans password entrer le mot de passe du serveur qui détient le cluster dans notre cas tout le mot de passe proxmox est `rftgy#123`. Il nous reste plus cas choisir le cluster network qu'il vous propose.

<img src ="img\Screen_ceph\Capture du 2022-12-12 15-51-16.png">
<br>
<br>

### <ins> 1.2.2.Ceph :</ins> 
#### <ins>Etape 1 :</ins>
<br>
Après avoir crée est rejoint notre cluster avec les deux autres serveurs il faut installer notre CEPH.
<br>
Pour celà il faut cliquer sur CEPH dans la catégorie de notre noeud numéro un, deux et trois. Puis cliquer sur install ( Attention en aucun cas ne fermer pas la page car votre installation risque de crash et vous devez tout recommencer). Une fois avoir cliquer sur l'install il se fera automatiquement seulement dans le premier noeud vous aurez a choisir votre `Public Network` qui est l'IP de votre serveur numéro un, pour les autres l'installation ce fait automatiquement.
<br>
<br>

<img src = "img\Screen_ceph\Capture du 2022-12-12 16-08-54.png">
<br><br>

#### <ins>Etape 2 :</ins>
<br>

Sur votre noeud dans la catégorie `CEPH` puis `Monitor` ajouter les moniteur et manager deux et trois en cliquant sur `Create` dans la partie Monitor ou Manager 
<br><br>
<img src ="img\Screen_ceph\Capture du 2022-12-12 16-30-27.png">
<img src ="img\Screen_ceph\Capture du 2022-12-12 16-30-16.png">


#### <ins>Etape 3 :</ins>

Sur chaque noeud il va falloir crée une OSD, donc depuis la catégorie `CEPH` puis `OSD` cliquer sur crée est sélectionnées votre partition de disque libre 

<img src= "img\Screen_ceph\Capture du 2022-12-13 15-47-25.png"><br>

Une fois fini : 
<img src= "img\Screen_ceph\Capture du 2022-12-13 15-50-27.png">


#### <ins>Etape 4:</ins>
Il faut maintenant crée une pool depuis le pve1, pour faire celà aller dans `CEPH` puis `Pools`, cliquer sur `Create`

<img src= "img\Screen_ceph\Capture du 2022-12-13 16-24-38.png"><br><br>

#### <ins>Etape 5:</ins> <br>
Pour voir le bon fonctionnement de toutes votre installation il faut vérifier l'état de santé de notre CEPH.
Pour cela il suffit de cliquer sur `CEPH` : 

<img src= "img\Screen_ceph\Capture du 2022-12-13 16-55-33.png">
<br><br><br><br>

## <ins>2.Utilisation :</ins>

#### <ins> 2.1 Création d'une VM</ins>

La création de VM est un peu particulier sur proxmox l'installation via ISO, ça nécessitent de retirer la carte réseaux et de le r'ajouter après l'installation mais aussi de retirer le CD/DVD après l'installation pour pouvoir migrer la VM.
<br><br>

Pour crée votre VM il faut d'abord upload votre iso, rendez-vous dans votre noeud et cliquer sur `Image ISO` et cliquer sur `UPLOAD`,puis `Select File` pour selectionner votre ficher, finir en cliquant sur `UPLOAD`

<img src="img\screen_create_vm\create_vm1.png">
<br><br>
Pour crée une VM, maintenant vous devez cliquer sur `Create VM` en haut à droite :<br>
Il vous faut choisir le noeud d'appartenance, une ID est donner par défauts changer la si vous le souhaitez, donner un nom à votre VM.
<img src="img\screen_create_vm\create_vm2.png">
<br><br>
Choissisez l'image ISO que vous souhaitez que votre VM doit prendre.
<img src="img\screen_create_vm\create_vm3.png">
<br><br>
Je laisse par défauts
<img src="img\screen_create_vm\create_vm4.png">
<br><br>

Définisser si vous voulez utiliser le stockage local ou ceph depuis `Storage` et la taille de Disk dans mon cas 8Go est largement suffisants
<img src="img\screen_create_vm\create_vm5.png">
<br><br>

Quelque chose d'important sur proxmox est que pour chaque VM vous pouvez limiter la bande passante seulement sur la VM sans passer par une limitation au niveau du port comme VMWare
<img src="img\screen_create_vm\create_vm6.png">
<br><br>
On peut augmenter le nombre de coeurs allouer
<img src="img\screen_create_vm\create_vm7.png">
<br><br>
On peut augmenter la RAM (mémoire vice) allouer (Valeur en Mo)
<img src="img\screen_create_vm\create_vm8.png">
<br><br>

Ici on doit crée une vm sans carte réseaux pour l'ajouter après l'installation ISO fini, car dans l'établissement les installations de paquets et autres nous prend beaucoup de temps à cause de la connection internet. Donc pour éviter les installations de paquets ou autres il vaut mieux désactiver la création de carte réseaux (je vous montre par la suite comment la crée). Cliquer sur ``No network device`.
<img src="img\screen_create_vm\create_vm9.png">
<br><br>

Vous pouvez relire les informations de création de votre VM pour vous assurer de votre configuration souhaitez.
<br>Si vous souhaitez le démarrage après la création cliquer sur `Start after created` (en bas à gauche)
<img src="img\screen_create_vm\create_vm10.png">
<br><br>

On peut observer qu'on a bien notre vm de crée mais il n'a pas de connection internet 
<img src="img\screen_create_vm\create_vm11.png">
<br><br>

Cliquer sur votre VM,ensuite dans le menu déroulant cliquer sur `Hardware`, puis sur `Add` et selectionner `Network Device`. Selectionner votre bridge et appuyer sur `Add`
<img src="img\screen_create_vm\create_vm12.png">
<br><br>

On vérifie bien en ping 8.8.8.8, on a bien à une connection internet.
<img src="img\screen_create_vm\create_vm13.png">

#### <ins> 2.1 Migration d'une VM</ins>

Pour migrer une VM après l'installation via ISO il necéssitent de se rendre dans la partie `Hardware` de votre VM et de modifier le `CD/DVD Drive` est de le mettre en mode `Do not use any media` 

<img src="img\screen_migrate\migrate1.png">
<img src="img\screen_migrate\migrate2.png">
<br><br>

<ins>Migration à froid :</ins>
<br><br>

Ici nous avons notre VM nommée "CentosMini" dans le noeud "pve1".

<img src="img\screen_migrate\migrate3.png">

<br><br>
On clique droit dessus puis sur `Migrate` et on choissie vers quel noeud on voeud le migrer (mauvais screen j'ai migrer vers le noeud 3 on pourra le voir dans le screen suivants)
<img src="img\screen_migrate\migrate4.png">
<br><br>

On peut observer le message (to node 'pve3', donc vers le noeud 3), la migration à réussis
<img src="img\screen_migrate\migrate5.png">
<img src="img\screen_migrate\migrate6.png">
<br><br>

<ins>Migration à chaud :</ins>

Maintenant, on va démarrer la VM pour faire une migration à chaud

<img src="img\screen_migrate\migrate7.png">
<br><br>
On migre au noeud de départ donc la une (pve1)

<img src="img\screen_migrate\migrate8.png">
<br><br>
On peut voir que la migration à encore réussis

<img src="img\screen_migrate\migrate9.png">
<img src="img\screen_migrate\migrate10.png">

#### <ins> 2.3 Création d'une template</ins>

A partir d'une VM nous pouvons crée une template en fessant clique droit `Convert to template`
<br>
<img src="img\screen_template\template1.png">
<br><br>
Pour déployer une VM depuis une template il faut faire clique droit dessus est choissir `Clone`, donner un nom a votre nouvelle VM et le mode de clonage par exemple Full Clone

<img src="img\screen_template\template2.png">
<br><br>
Le tour est jouer vous avez crée votre VM
<br>

<img src="img\screen_template\template3.png">


---
#### <ins> 2.4 Commande qui mon était utiles</ins>

Cette partie regroupe les commande que j'ai du utiliser lors de cette SAE pour proxmox 

Pour supprimer un cluster :

```
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

``` 
rm -r /etc/pve/[node_name]
```

Pour modifier l'IP du serveur proxmox :

```
vi /etc/network/interfaces
systemctl restart networking.service
```