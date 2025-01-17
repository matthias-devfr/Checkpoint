# Exercice 1 : Manipulations pratiques sur VM Windows (temps estimé : 1h30)

## Partie 1 : Gestion des utilisateurs
L'utilisateur Kelly Rhameur a quitté l'entreprise.
Elle est remplacée par Lionel Lemarchand

Q.1.1.1 Créer l'utilisateur Lionel Lemarchand avec les même attribut de société que Kelly Rhameur.  
Dans 'Active Directory Users and Computers' > Action > Find... rechercher 'Kelly Rhameur' > Clic droit sur Kelly Rhameur > copy
Une fenetre s'ouvre
Entrer le first name, last name et user logon name de Lionel Lemarchand.

Q.1.1.2 Créer une OU DeactivatedUsers et déplace le compte désactivé de Kelly Rhameur dedans.  
Clic droit sur l'OU 'LabUsers' > new > organizational unit > Entrer le nom de l'OU 'DeactivatedUsers'
Clic droit sur Kelly.Rhameur > 'Disable Account'
Pour la déplacer maintenir clic gauche sur Kelly.Rhameur et glisser dans l'OU 'DeactivatedUsers'

Q.1.1.3 Modifier le groupe de l'OU dans laquelle était Kelly Rhameur en conséquence.  
Double clic gauche sur Kelly Rhameur > member of > selectionner le groupe 'GrpUsersDirectionDesRessourcesHumaines' et cliquer sur 'remove'

Q.1.1.4 Créer le dossier Individuel du nouvel utilisateur et archive celui de Kelly Rhameur en le suffixant par -ARCHIVE.  
Dans 'file explorer' > DossiersIndividuels (F:) clic droit sur kelly.rhameur > rename et rajouter '-ARCHIVE' à la fin.
Dans 'file explorer' > DossiersIndividuels (F:) clic droit > new folder > entrer 'lionel.marchand'

## Partie 2 : Restriction utilisateurs
Q.1.2.1 Faire en sorte que l'utilisateur Gabriel Ghul ne puisse se connecter que du lundi au vendredi, de 7h à 17h.  
Retour dans 'Active Directory Users and Computers' > Action > Find... rechercher 'Gabriel Ghul'
Clic droit sur son compte > Properties > Account
Cliquez sur le bouton Logon Hours.
Dans la fenêtre qui s'ouvre sélectionnez tous les créneaux en dehors de lundi à vendredi, de 7h à 17h (Samedi et Dimanche compris) et cliquez sur Deny.

Q.1.2.3 Mettre en place une stratégie de mot de passe pour durcir les comptes des utilisateurs de l'OU LabUsers.  
Dans 'Group Policy Management' clic droit sur 'Group Policy Object' > new
Entrez le nom de la GPO 'lab_users-security-password_policy'
Clic droit dessus > edit
Aller dans : Computer Configuration > Policies > Windows Settings > Security Settings > Account Policies > Password Policy
Enforce password history : 10.
Maximum password age : 42
Minimum password length : 12
Password must meet complexity requirements : Enabled
- Lier la GPO à l'OU
Clic droit sur l'OU 'LabUsers' > Link an Existing GPO. selectionner la GPO 'lab_users-security-password_policy'

# Exercice 2 : Manipulations pratiques sur VM Linux (temps estimé : 2h30)
Pour cet exercice tu as besoin de la VM SRVLX01.

## Partie 1 : Gestion des utilisateurs
Q.2.1.1 Sur le serveur, créer un compte pour ton usage personnel.  
```bash
sudo adduser matt
```

Q.2.1.2 Quelles préconisations proposes-tu concernant ce compte ?  
l'ajouter au groupe sudo
```bash
usermod -aG sudo matt
```

## Partie 2 : Configuration de SSH
Un serveur SSH est lancé sur le port par défaut.
Il est possible de s'y connecter avec n'importe quel compte, y compris le compte root.

Q.2.2.1 Désactiver complètement l'accès à distance de l'utilisateur root.  
```bash
sudo nano /etc/ssh/sshd_config
```
ajouter :
```
PermitRootLogin no
```

Q.2.2.2 Autoriser l'accès à distance à ton compte personnel uniquement.  
```bash
sudo nano /etc/ssh/sshd_config
```
ajouter :
```
AllowUsers matt
```

Q.2.2.3 Mettre en place une authentification par clé valide et désactiver l'authentification par mot de passe  
```bash
sudo nano /etc/ssh/sshd_config
```
ajouter :
```
PubkeyAuthentification yes
PasswordAuthentification no
```


## Partie 3 : Analyse du stockage
Q.2.3.1 Quels sont les systèmes de fichiers actuellement montés ?  
```bash
blkid
```
il y a :
- udev
- tmpfs
- /dev/mapper/cp3--vg-root (ce qui indique un VolumeGroupe)
- /dev/md0p1 (Ce qui indique une partition RAID)

Q.2.3.2 Quel type de système de stockage ils utilisent ?  
Il y a du SWAP, ext2, ext4, LVM2_member et linux_raid_member

Q.2.3.3 Ajouter un nouveau disque de 8,00 Gio au serveur et réparer le volume RAID  
Eteindre la VM et créer un disque de 8Go, redémarrer.
```bash
lsblk #pour trouver le disque
fdisk /dev/sdb # n (nouvelle partition > p (primaire) > 3x entrer > t (type de partition) > fd (LinuxRaid) > w (ecrire la partition))
lsblk # pour verifier que la partition est bien sdb1
mdadm --detail /dev/md0 #pour verifier l'état du raid
mdadm --manage /dev/md0 --add /dev/sdb1 # attendre que le raid se synchronise
mdadm --detail /dev/md0 # verifier que le raid est actif
```

## Partie 4 : Sauvegardes
Le logiciel bareos est installé sur le serveur.
Les composants bareos-dir, bareos-sd et bareos-fd sont installés avec une configuration par défaut.

Q.2.4.1 Expliquer succinctement les rôles respectifs des 3 composants bareos installés sur la VM.  
bareos-dir : Orchestrateur de la sauvegarde, il envoie les instructions aux autres composants bareos-sd et bareos-fd.
bareos-sd : Il gère les ressources de stockage utilisées pour les sauvegardes
bareos-fd : IL sauvegarde et restaure les fichiers sur les clients.


## Partie 5 : Filtrage et analyse réseau
Q.2.5.1 Quelles sont actuellement les règles appliquées sur Netfilter ?  
```bash
nft list ruleset
```
il accepte : 
- le protocole TCP sur le port 22
- le protocole ICMP est accepté pour l'IPv4 et IPv6

Q.2.5.2 Quels types de communications sont autorisées ?  
les connexions SSH et le ping IPv4/6 sont autorisés

Q.2.5.3 Quels types sont interdit ?  
Je ne vois aucune conexion rejetée mise a part 'ct state ivalid drop'

Q.2.5.4 Sur nftables, ajouter les règles nécessaires pour autoriser bareos à communiquer avec les clients bareos potentiellement présents sur l'ensemble des machines du réseau local sur lequel se trouve le serveur.  
Rappel : Bareos utilise les ports TCP 9101 à 9103 pour la communication entre ses différents composants.  
J'ai dabord créé une table 'bareos_filter'
```bash
nft add table inet bareos_filter
```
puis j'ai ajouté la chaine input a bareos_filter
```bash
nft add chain inet bareos_filter input { type filter hook input priority 0\; }
```
j'ai ajouté la chaine output a bareos_filter
```bash
nft add chain inet bareos_filter output { type filter hook input priority 0\; }
```
et j'ai finalement réussi a ajouter les règles d'entrée et de sortie (après de nombreux échec!)
```bash
nft add rule inet bareos_filter input tcp dport {9101,9102,9103}
nft add rule inet bareos_filter output tcp sport {9101,9102,9103}
```
## Partie 6 : Analyse de logs
Q.2.6.1 Lister les 10 derniers échecs de connexion ayant eu lieu sur le serveur en indiquant pour chacun :

La date et l'heure de la tentative
L'adresse IP de la machine ayant fait la tentative
```bash
journalctl | grep "authentication failure" | tail -n 10
```
