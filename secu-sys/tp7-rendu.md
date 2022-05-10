# TP 7 - Gestion des identités et des accès

## Intro

 - *L’ANSSI recommande la protection du chargeur de démarrage par un mot de passe. Pensez vous que cette mesure soit adaptée à votre cas ? Justifiez votre réponse*

Dans notre cas, on veut appliquer la defense en profondeur sur un serveur linux. C'est pour cela que nous avons besoin de mettre le plus de sécurité possible, je pense en effet qu'un mot de passe au démarrage pourra être utile dans notre cas.

## 3.1.2 Sécurisation de l’administration du serveur

 - *Quelles bonnes pratiques l'administrateur doit mettre en place pour sécuriser sa biclef ?*

En se basant sur les recommandations de l'ANSSI, voilà ce qu'il devra faire : 
 - Clef ECDSA
 - Clef de 256 bits
 - Les clés doivent être générées dans un contexte où la source d’aléa est fiable, ou à défaut dans un environnement où suffisamment d’entropie a été accumulée.


### Politique de sécurité

Au niveau de la politique de sécurité, plus précisement des mots de passe, voilà ce que nous allons mettre en place : 
 - Longueur minimal : 10
 - Une majuscule
 - Une minuscule
 - Une ponctuation
 - Un chiffre

Pour se faire, on va venir modifier ce fichier `/etc/pam.d/common-password`

![](https://i.imgur.com/c2azAyt.png)

Et voilà ce qu'on y ajoute, maintenant, si je change mon mot de passe et que je mets "test123" voilà l'erreur qui sort :

![](https://i.imgur.com/liDovT1.png)


## Configuration et durcissement du rôle serveur de fichiers

### SAMBA

On installe Samba : `sudo apt install samba`

On vient créer notre dossier de fileshare :  `mkdir /srv/fileshare`

On vient ajouter nos ligne au fichier de conf : `/etc/samba/smb.conf`

![](https://i.imgur.com/XDOtvUz.png)

 - *Est-il nécessaire de chiffrer cette partition ? Justifiez votre réponse.*

Oui, il est nécessaire de chiffrer cette partition, c'est notre serveur de partage de fichiers, les fichiers seront potentiellement sensible.

### Comptes utilisateurs

Script : 

```shell=#!/usr/bin/bash
# AdrienIT
# 09/05/2022

# This script takes 2 inputs, the first one are the users and the second one are the groups
# For multiple user or/and multiple groups, you can do it with space -> user1,user2 -- group1,group2

echo "Please write the user you want to create"

read -a Users

echo "Are you sur you want to create those users ? (y/n) "
read choice

if [[ "$choice" ==  [yY] ]]
then
    for each in "${Users[@]}"
    do
      useradd "$each"
      password="$(gpg --gen-random --armor 1 14)"
      echo "$each:$password" | chpasswd
      echo "$each created successfully"
   done
fi

echo "=== USERS DONE ==="

echo "Please write the groups you want to create"

read -a Groups

echo "Are you sur you want to create those groups ? (y/n) "
read choice2

if [[ "$choice2" ==  [yY] ]]
then
    for each in "${Groups[@]}"
    do
      groupadd "$each"
      echo "$each created successfully"
   done
fi

echo "=== GROUPS DONE ==="
echo "Exiting ..."
```

Et voilà le résultat : 
![](https://i.imgur.com/EMFhAuz.png)


## Préparation de la structure des répertoires

Création de nos users et de nos groupes

![](https://i.imgur.com/4FlUZMi.png)

![](https://i.imgur.com/55O4k2W.png)


Ajouts dans nos users dans les groups

![](https://i.imgur.com/XyRV3EO.png)

Voilà la structure de notre fileshare
![](https://i.imgur.com/QEMc6tW.png)

On va maintenant assigner les droits aux différents users/groups :)

En commençant par les groupes

(résultat obenu avec un history | grep "setfacl")

```
  164  sudo setfacl -m g:direction:rw- direction/
  166  sudo setfacl -m g:pilotage:r-- direction/stategie/ 
  167  sudo setfacl -m g:srv.compta:r-- direction/stategie/ 
  168  sudo setfacl -m g:srv_compta:r-- direction/stategie/ 
  169  sudo setfacl -m g:srvcompta:r-- direction/stategie/ 
  170  sudo setfacl -m g:srvcompta:r direction/stategie/ 
  171  sudo setfacl -m g:srv.compta:r direction/stategie/ 
  172  sudo setfacl -m g:srv.compta:r direction/stategie 
  174  sudo setfacl -m g:srv.compta:r stategie/ 
  184  sudo setfacl -m g:pilotage:r direction/stategie/ 
  185  sudo setfacl -m g:srv.comptable:r direction/stategie/ 
  186  sudo setfacl -m g:direction:rw direction/achats/ 
  187  sudo setfacl -m g:direction:r compta/ 
  188  sudo setfacl -m g:direction:r compta/compta/ 
  189  sudo setfacl -m g:direction:r compta/facuration/
  190  sudo setfacl -m g:direction:r compta/tva/
  191  sudo setfacl -m g:direction:r administratif/
  192  sudo setfacl -m g:direction:rw administratif/gestion_conge/
  193  sudo setfacl -m g:direction:rw administratif/gestion_forma_interne/
  194  sudo setfacl -m g:direction:rw administratif/qualite/
  195  sudo setfacl -m g:direction:rw administratif/achats/
  196  sudo setfacl -m g:direction:r dsi
  197  sudo setfacl -m g:direction:r dsi/procedures/
  198  sudo setfacl -m g:direction:r dsi/divers_scripts/
  199  sudo setfacl -m g:direction:r dsi/docu_si/
  200  sudo setfacl -m g:direction:r depot/
  201  sudo setfacl -m g:direction:r commun/
  202  sudo setfacl -m g:direction:rw commun/direction/
  203  sudo setfacl -m g:direction:r commun/administratif/
  204  sudo setfacl -m g:direction:r commun/informatique/
  205  sudo setfacl -m g:direction:r commun/production/
  206  sudo setfacl -m g:pilotage:r direction/stategie/
  207  sudo setfacl -m g:pilotage:r administratif/gestion_forma_interne/
  208  sudo setfacl -m g:pilotage:r administratif/qualite/
  209  sudo setfacl -m g:pilotage:r administratif/achats/
  210  sudo setfacl -m g:srv.comptable:r direction/achats/ 
  211  sudo setfacl -m g:srv.comptable:r compta/compta/
  212  sudo setfacl -m g:srv.comptable:r compta/tva/
  213  sudo setfacl -m g:srv.comptable:r compta/facuration/
  214  sudo setfacl -m g:srv.comptable:r administratif/achats/
  215  sudo setfacl -m g:srv.comptable:r commun/direction/
  216  sudo setfacl -m g:srv.comptable:r commun/administratif/
  217  sudo setfacl -m g:srv.comptable:r commun/informatique/
  218  sudo setfacl -m g:srv.comptable:r commun/production/
  219  sudo setfacl -m g:srv.info:rw dsi
  220  sudo setfacl -m g:srv.info:rw dsi/procedures/
  221  sudo setfacl -m g:srv.info:rw dsi/divers_scripts/
  222  sudo setfacl -m g:srv.info:rw dsi/docu_si/
  223  sudo setfacl -m g:srv.info:r commun/direction/
  224  sudo setfacl -m g:srv.info:r commun/administratif/
  225  sudo setfacl -m g:srv.info:r commun/informatique/
  226* sudo setfacl -m g:srv.info:r
  227  sudo setfacl -m g:srv.logistique:rw depot/
  228  sudo setfacl -m g:srv.logistique:r commun/direction/
  229  sudo setfacl -m g:srv.logistique:r commun/administratif/
  230  sudo setfacl -m g:srv.logistique:r commun/informatique/
  231  sudo setfacl -m g:srv.logistique:r commun/production/
```

Et pour nos users  : 

![](https://i.imgur.com/oDCVnS7.png)

Enfin, pour fair en sorte que seul le propriétaire du fichier puisse le supprimer, il faut faire `chmod o+t -R .`

![](https://i.imgur.com/kFdfjnl.png)


### Sauvegarde du serveur

Voilà le script, je pense qu'il est assez bien commenté pour que ce soit compréhensible.

```shell=bash
#!/usr/bin/bash
# AdrienIT

# 10/05/2022

# Ce script nous permet de sauvegarder le repertoire /srv/fileshare. Il doit etre lancé avec le user SauvegardeServeur. L'archive doit etre chiffrée en aes-256-gcm et la clef de chiffrement est envoyé sur un serveur discord privé.
# Ce script est lancé tous les jours à 7h du matin. 

### COLORS ###

# Colors :D
declare -r NC="\e[0m"
declare -r B="\e[1m"
declare -r RED="\e[31m"
declare -r GRE="\e[32m"

### VARIABLES ###

# Target directory : the one we want to backup
declare -r target_path="${1}"
declare -r target_dirname=$(awk -F'/' '{ print $NF }' <<< "${target_path%/}")

# Craft the backup full path and name
declare -r backup_destination_dir="/opt/backup/"
declare -r backup_date="$(date +%y%m%d_%H%M%S)"
declare -r backup_filename="${target_dirname}_${backup_date}.tar.gz"
declare -r backup_destination_path="${backup_destination_dir}/${backup_filename}"

declare -r discordurl="https://discord.com/api/webhooks/942802933551083520/h07MxxPvpwGzAD28z-2IJ4XHDbfV9mk5z5km0hdQcxKG7OzrLTk7hjnhn8jzczzUFn3S"
declare -r randkey="$(gpg --gen-random --armor 1 14 | base64)"

### FUNCTIONS ###

# Get timestamp in order to log
get_current_timestamp() {
  timestamp=$(date "+[%h %d %H:%M:%S]")
}

# Echo arguments with a timestamp
log() {
  log_level="${1}"
  log_message="${2}"

  get_current_timestamp

  if [[ "${log_level}" == "ERROR" ]]; then
    echo -e "${timestamp} ${B}${RED}[ERROR]${NC} ${log_message}" >&2

  elif [[ "${log_level}" == "INFO" ]]; then
    echo -e "${timestamp} ${B}[INFO]${NC} ${log_message}"

  fi
}


# Craft an archive, compress it, and store it in $backup_destination_dir
archive_and_compress() {

  dir_to_backup="${1}"

  log "INFO" "Starting backup."

  # Actually creates the compressed archive
  tar cvzf \
    "${backup_destination_path}" \
    "${dir_to_backup}" \
    --ignore-failed-read &> /dev/null

  # Ciphering the file
  openssl aes-256-cbc -a -salt -pbkdf2 -in "${backup_destination_dir}""${backup_filename}" -out "${backup_destination_dir}""${backup_filename}.enc" -k "${randkey}"

# Test if the archive has been created successfully
  if [[ $? -eq 0 ]]
  then
    log "INFO" "${B}${GRE}Success.${NC} Backup ${backup_filename} has been saved to ${backup_destination_dir}."
    sendkey='{"username": "Bot - key receiver", "content": "Uncipher key : '"$randkey"'"}'
    curl -s -H "Content-Type: application/json" -H 'accept: application/json' -d "$sendkey" $discordurl
    curl -s -H 'accept: application/json' -H 'accept-encoding: gzip, deflate, br' -H 'content-type: multipart/form-data' -F "file=@${backup_destination_dir}""${backup_filename}.enc" $discordurl
    rm -f "${backup_destination_dir}""${backup_filename}"
  else
    log "ERROR" "Backup ${backup_filename} has failed."

    # Even if tar has failed, it creates the archive, so we remove it in case of failure
    rm -f "${backup_destination_path}"

    exit 1
  fi
}

# DO THE MAGIC
archive_and_compress "${target_path}"
```

Au niveau de l'execution, voilà ce que ça donne : 
![](https://i.imgur.com/tOc2LcU.png)

et au niveau des resultats : 
![](https://i.imgur.com/IXTVkgc.png)

Et on reçoit bien la clef de déchiffrement sur discord : 
![](https://i.imgur.com/6Bbxmj0.png)

C'est bien évidemment, la dernière :)

Il faut savoir que j'ai aussi ajouté la crontab pour executer ce script tous les jours à 7h.

On donne aussi l'accès qu'aux membres du groupe srv.info et on met root en owner : 
![](https://i.imgur.com/KRao7ux.png)


## Revue des accès et contrôle de conformité

Pour checker les users voilà comment on procéde, il y a 3 cas : 
 - Si le user est présent dans la liste qui est envoyé et si il est aussi présent sur le systeme
 - Si il est present dans la liste mais pas sur le systeme
 - SI il est present sur le systeme mais pas dans la liste (Désactivation)

Pour se faire, j'ai rédigé un script qui va venir faire ce check en fonction des input : 

```shell=bash
#!/usr/bin/bash
# AdrienIT
# 10/05/2022

# Ce script permet de voir si un nouvel utilisateur a été ajouté ou si un utilisateur n'est plus présent dans la matrice de reférence.

# variables

declare -r NC="\e[0m"
declare -r B="\e[1m"
declare -r RED="\e[31m"
declare -r GRE="\e[32m"
usersnext=( $(getent passwd | cut -d ":" -f1 | grep \\. ) )

# functions

get_current_timestamp() {
  timestamp=$(date "+[%h %d %H:%M:%S]")
}

log() {
  log_level="${1}"
  log_message="${2}"

  get_current_timestamp

  if [[ "${log_level}" == "ERROR" ]]; then
    echo -e "${timestamp} ${B}${RED}[ERROR]${NC} ${log_message}" >&2

  elif [[ "${log_level}" == "INFO" ]]; then
    echo -e "${timestamp} ${B}[INFO]${NC} ${log_message}"

  fi
}


# code

echo "===== CHECKING USERS CHANGE ====="

echo "Please write the user you want to checks, including new ones (y/n) : "
read -a Users

echo "Are you sur ? (y/n) "
read choice


# check if the user is new or if the users is still active
if [[ "$choice" ==  [yY] ]]
then
    for each in "${Users[@]}"
    do
      if getent passwd "$each" > /dev/null 2>&1; then
        log "INFO" "${B}${GRE}$each${NC} is still active"
        getfacl -R -p /srv/fileshare | grep "user" | grep "$each" | cut -d ":" -f3 | tr '\n' ',' > work.csv
      else
        log "INFO" "$each is new in the users list"
      fi
   done
fi

# check if account is in the system but not from the user input
for each in "${usersnext[@]}"
do
        if printf '%s\0' "${Users[@]}" | grep -Fxqz "$each"; then
                :
        else
                log "ERROR" "${B}$each${NC} is now set as inactive, it will be dsabled ..."
                chage -E0 "$each"
        fi
done

echo "===== CHECKING RIGHT CHANGE ====="
# from user input
# get all files from /srv/fileshare

getfiles=( $(getfacl -R -p /srv/fileshare | grep "file" | awk '{print $3}') )

echo "username,/srv/fileshare,/srv/fileshare/direction,/srv/fileshare/direction/stategie,/srv/fileshare/direction/achats,/srv/fileshare/administratif,/srv/fileshare/administratif/qualite,/srv/fileshare/administratif/achats,/srv/fileshare/administratif/gestion_forma_interne,/srv/fileshare/administratif/gestion_conge,/srv/fileshare/commun,/srv/fileshare/commun/production,/srv/fileshare/commun/direction,/srv/fileshare/commun/informatique,/srv/fileshare/commun/administratif,/srv/fileshare/compta,/srv/fileshare/compta/tva,/srv/fileshare/compta/compta,/srv/fileshare/compta/facuration,/srv/fileshare/depot,/srv/fileshare/dsi,/srv/fileshare/dsi/divers_scripts,/srv/fileshare/dsi/docu_si,/srv/fileshare/dsi/procedures" > final.csv

for each in "${getfiles[@]}"
do
        #log "INFO" "Trying to get right on : $each"
        for input in "${Users[@]}"
        do
                getfacl -t -p "$each" | grep "$input" &> /dev/null
                if [ $? == 0 ]; then
                        right=$(getfacl -t -p "$each" | grep "$input" | awk '{print$3}')
                fi
        done
done

```

Par exemple si le PDG décide de virer tout le monde, et de m'ajouter que moi voila ce qu'il va se passer : 

![](https://i.imgur.com/lZVDz8e.png)

On remarque donc que ef.wagner est toujours actif, que le user adrien.z  va etre ajouté et que tous les autres users vont être désactivé.

J'ai aussi essayé de faire la matrice de droit mais c'était bien trop compliqué pour moi. Beaucoup de tentatives et beaucoup d'echec, on peut voir le fichier final.csv sur lequel j'allais me baser pour créer cette matrice mais je n'arrivai pas à ajouter les droits en fonction de chaque users et en fonction de chaque fichiers dans mon CSV