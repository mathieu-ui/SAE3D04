# Situation Pro

Etude de virtualisation en remplacement de VMware.

Ex : Proxmox (ceph), HyperV, Open Nebula.

Argumenter :

- Tableau
- Comparer (stockage)
- Comparer le resaux, avec des shemat, decire les architecture,
- Bilan, diagrame de gante
- Tache, Jira
- Preuve
- comparaison des couts
- Securité

## Le Groupe

- Julien Alleaume
- Ilker Onay
- Mathieu Puig
- Ndeye Codou Touré

## Premiere Instalation

### 3 proxmox

- On fait en premier l'installation de deux proxmox qui seront par la suite completer par un troisieme serveur pour faire un systeme CEPH

### 2 hyperV

- Ici vas installer deux serveur HyperV, afin de raliser une comparaison entre les deux.

## Windows utils

```bash
Get-WindowsCapability -Online | ? Name -like 'OpenSSH*'
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Start-Service sshd
Get-Service sshd
Set-Service -Name sshd -StartupType 'Automatic'
```
