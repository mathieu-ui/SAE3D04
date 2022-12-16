# Proxmox vs VMware


Tout deux sont des solutions de virtualisations, mais quel est le meilleur ?

Deux choix s'offrent à vous donc pour celà nous allons prendre en compte trois grande ligne de comparaison :
<br> -Le coût
<br> -Flexibilité et facilité d'utilisation
<br> -Les performances

<br><br>

## <b><ins>Proxmox :</ins></b>

Proxmox est une licence open source baser sur Debian Linux KVM, vous avez une version d'essai gratuit illimité (celui que nous avons utiliser), ou des versions tarifaire qui varie de 104$ à 975$ /an par Socket CPU

## <b><ins>VMWare :</ins></b>

VMWare lui est une licence propriétaire baser sur VMkernel, vous avez une version d'essai gratuit de 60 jours, ou des versions tarifaire qui varie de 578$ à 5596$ /an par Socket CPU
<br><br>

<img src="img\screen_comparaison\1.png">

### <ins><b>Le prix :</ins></b>

Si votre budget et limité (indépendant ou curieux) il serait plus préférable de choisir une version proxmox 

On peut voir ci-dessous la forte différence entre les prix d'une licence Proxmox et VMWare (la plus chère licence dans les 2 cas)


<img src="img\screen_comparaison\prix.png">

<br><br>

On peut aussi comparer les prix des licences un peu moin chère des deux côters.
<br>
<ins>Proxmox : </ins>
<br>
<img src="img\screen_comparaison\2.1.png">
<br>
<ins>VMWare : </ins>
<br>
<img src="img\screen_comparaison\2.png">
<br>

### <b><ins>La flexibilité et facilité d'utilisation :</ins></b>

VMware est beaucoup plus flexible et facile à utiliser, tant dis que sur Proxmox il faut parfois passer par des lignes  de commandes obligatoirement, par exemple pour supprimer noeuds on est obliger de passer par la commande :

```
rm -r /etc/pve/[node_name]
```
<br>
Tandis que sur VMWare un simple click droit suffit sur le noeud.
<br><br>
<ins>Autres exemple :</ins><br>
-Joindre les noeuds proxmox posent problème si les noms de noeuds correspond, tant dis que VMWare s'appuie sur les IP<br>
-Supprimer, pour supprimer la moindre chose faut cliquer dessus et aller chercher dans l'onglet déroulant "Bulk Action" et choissir l'options "Remove", tant dis que sur VMWare clique droit puis supprimer.<br>
-Crée une VM via OVA n'est pas possible directement sur Proxmox<br>
-Crée une VM via un ISO sur le PC n'est pas possible il faut l'upload sur le serveur d'abord<br>
-ect,ect,..<br><br>

Interface graphique est beaucoup plus développez sur VMWare que sur Proxmox, même au niveaux des données que se soit des utilisations de CPU, Mémoire, ect...


### <b><ins>Les performances : </ins></b>

Une différence énorme ce fait ressentir lors des démarrages de VM sur VMWare et Proxmox ou encore lors de migrations.

<ins>On peut les observers avec ce graph :</ins>

On peut peut observer qu'arrêter, démarrer ou migrer une vm prend plus de temps sur proxmox. <br>
<br>
<img src="img\screen_comparaison\perf2.png">



Vous devez aussi savoir que VMWare propose une utilisation maximum de RAM ,d'hôtes par Cluster et de processeurs beaucoup plus élevées
<br>

<img src="img\screen_comparaison\perf.png">

<br><br>

### <b><ins>Conclusion : </ins></b>

Si vous avez pas de budget ou très peu il peut être intéréssants de prendre un Proxmox, mais si votre entreprise ou vous pour votre utilisation personnel vous avez un budget très ouvert il peut être plus appréciable et agréable d'utiliser VMWare pour plus de performances et de facilités.