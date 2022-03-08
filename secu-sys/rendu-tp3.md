# TP 3 : HIDS - AIDE

## 3.1 : Découverte de l'outil

### 3.1.2 : Rappels théoriques

Un HIDS, de son nom Host-Based Detection System, est un service qui permet de detecter des actions anormales et/ou de detcter des actions malveillantes sur un système. 

Quant à l'IDS, de son nom Instrusion Detection System, permet lui aussi de detecter des actions anormales mais plus basé sur le réseau et non sur le systeme comme le ferait un HIDS. Ces 2 outils sont donc complémentaires !

AIDE utilise des systèmes de cryptographie afin de générer une empreinte à un instant T et de vérifier qu'elle ne bouge pas à l'instant T+1. Pour générer ces empreintes, AIDE utilise des algorithmes de hachage et de chiffrment.

### 3.1.3 : Fonctionnement de AIDE

AIDE (Advanced Intrusion Detection Environnement) est un outil qui permet de vérifier si des fichiers ont été modifiés via leur intégrité.

AIDE marche avec 2 grandes étapes.
La première, c'est de créer une base de données en se basant sur ces élements : 
 - file type
 - permissions 
 - inode number
 - user
 - group
 - file size
 - mtime and ctime
 - atime
 - growing size
 - number of links and link name.

Ensuite, AIDE va aussi générer un hash pour chaque fichier présent sur le système, voici la liste des algorithmes utilisés : 
 - sha1
 - sha256
 - sha512
 - md5
 - rmd160
 - tiger
 - haval
 - crc32

Ces 2 étapes vont générer une base solide et concrète sur ce qui existe sur le système. La moindre altération d'un des fichiers sera vu par AIDE.


## 3.2 : Configuration de l'outil

### 3.2.1 : Préparation du fichier de configuration

AIDEINIT 
![](https://i.imgur.com/SLpo81T.png)

On remplace l'ancienne BDD avec celle que l'on vient de générer : 
```
cp /var/lib/aide/aide.db.new /var/lib/aide/aide.db
```

AIDE CHECK : 
```
aide -c /etc/aide/aide.conf -C
```

Pour vérifier que notre configuration marche bien, on va venir ajouter un utilisateur : 
```
useradd john
```

Et voici le résultat de la commande `aide -c /etc/aide/aide.conf -C` 
![](https://i.imgur.com/BjKRHFC.png)

On va se pencher sur /etc/passwd

![](https://i.imgur.com/vijptnB.png)

On remarque que les hash ont changés et que la size a augmenté !

### 3.2.2 : Personnalisation de la configuration

Tout d'abord on va venir créer une règle pour checker si le fichier à été modifier

Voila la règle que l'on ajoute dans `/etc/aide/aide.conf`
![](https://i.imgur.com/gJw0b6z.png)

s pour checker la SIZE
sha512 pour checker si le hash à changer.
J'ai préferer ajouter le hash, car la size pour être identique mais le code altéré, le hash me donnera une confirmation.

Ensuite, on va venir créer un fichier `/etc/aide/aide.conf.d/onlyhttp` qui va être la regle pour vérifier notre fichier index.html, voila ce qu'il contient : 

![](https://i.imgur.com/ZWZyCtY.png)

On vient lui donner le fichier que l'on veu monitorer, ici `/var/www/html/index.html`
On vient lui indiquer le type, avec `f`, un fichier dans notre cas.
Et pour finir on lui donne la règle qu'il doit appliquer !

Après avoir fait ces modifs, nous allons créer une nouvelle base de donnée : 

![](https://i.imgur.com/2wGwePC.png)

Pour vérifier si notre conf marche bien, on vient modifier le fichier index.html : 

![](https://i.imgur.com/GS29DVc.png)

Et maintenant, mettons dans le cas où le sysadin s'est rendu compte que son site à été piraté, il a plus qu'a faire un check avec cette commande : `aide -c /etc/aide/aide.conf -C`

Et voilà le résultat : 

![](https://i.imgur.com/Z0jkjNZ.png)

On voit bien le fichier qui a été changé.
On voit aussi la size qui est passé de 6 à 9
Et forcément, le SHA512 qui a changé ! 

Notre règle marche bien !