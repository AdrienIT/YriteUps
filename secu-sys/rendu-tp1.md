# TP 1  -  Manipulation des grands principes de la cryptographie

### 3.1.1 - Rappels théoriques

#### Questions

1. Expliquez moi à quoi peut servir une fonction de hachage ? 
2. Dans le cas du téléchargement d'un fichier sur un site internet, quel risque cela permet-il de prévenir ?

#### Réponses

1. Une fonction de hachage est ne fonction qui calculer une empreinte unique à partir des données fournies. Ces fonctions sont très utiles car elles permettent l'unicité et l'authenticité des fichiers fournies. On peut dire que si 2 empreintes sont uniques alors les données fournies sont identiques et n'ont pas été altérés
2. Dans le cas d'un téléchargement de fichier sur un site internet, il faudrait que le site nous donne aussi le hash associé au fichier. Cela nous permetterai, une fois téléchargé, de calculer l'empreinte de ce dernier afin de garantir son authenticité et son intégrité. On prévient donc le risque de modification de fichier et par conséquent de réduire le risque que ce fichier contienne un malware.

### 3.1.2 - Contrôle d'intégrité d'un téléchargement

Téléchargement de gnupg : 

`wget https://gnupg.org/ftp/gcrypt/gnupg/gnupg-2.2.34.tar.bz2`

Sur le site de GNUpg, ils donnent les hashs SHA1 sur cette page https://gnupg.org/download/integrity_check.html

On va venir générer l'empreinte de notre fichier avec cette commande : 

```
openssl dgst -sha1 gnupg-2.2.34.tar.bz2 
SHA1(gnupg-2.2.34.tar.bz2)= b931cc1aa287ad67b0efacb91e7b358bf4852278
```

On remarque que notre input hashé vaut  : b931cc1aa287ad67b0efacb91e7b358bf4852278

En faisant un CTRL + F Sur le site que j'ai donné un peu plus haut on remarque que l'empreinte correspond bien et que le fichier n'a donc pas été altéré.


## Chiffrement de fichier à l'aide du chiffrement symétrique AES

### 3.2.1 - Grands principes du chiffrement symétrique

Le chiffrement symétrique est une méthode de chiffrement dans laquelle une seule et même clef est reponsable du cryptage et du décryptage des données fournies.
La méthode de chiffrement la lus utilisée est la méthode de chiffrement par bloc. Cette méthode de chiffrement consiste à découper un mesage en taille de blocs fixes puis de chiffer chaques blocs avec une clé secrète et une fonction de chiffrement. On vient ensuite regrouper tous les blocs pour créer le texte et/ou le fichier chiffré. 


### 3.2.2 -  Chiffrement de fichier 

Création d'un fichier avec `salut c'est adrien` dedans. 

On vient créer nos clefs / ivs avec cette commande : 

```
➜ openssl enc -aes-256-cbc -k abc -P -md sha1
*** WARNING : deprecated key derivation used.
Using -iter or -pbkdf2 would be better.
salt=326E93F02FA05C75
key=2E9F0C0A912C66C8A55B1CEF92DAB9CB0742B327AAD61196631B8012FE6A6C2D
iv =2BC8EA070F5CC1F4E9D92C32D6089F0E
```

On va venir le chiffrer avec la commande suivante

**ENCRYPTION**
`➜ openssl enc -aes-256-cbc -nosalt -e -in fichier1.txt -out fichier1.txt.enc -K '0AF07D08EC9EAF74DA770F5BF89738EA38F4DF32B5889C460F8870CFD24CCB88' -iv '736DAFB67515983D62C574A5AF989E8C'`

Voila ce qu'on a en essayer de lire le contenu du fichier : 
```
➜ cat fichier1.txt.enc 
mmX Pb+8`#(O*c%
```


Je donne ensuite le fichier `fichier1.txt.enc` la clef et l'iv à mon voisin et il va essayer de déchiffrer le fichier

**DECRYPTION**
```
➜ openssl enc -aes-256-cbc -nosalt -d -in fichier1.txt.enc -K 0AF07D08EC9EAF74DA770F5BF89738EA38F4DF32B5889C460F8870CFD24CCB88 -iv 736DAFB67515983D62C574A5AF989E8C
salut c'est adrien
````

## 3.3 - Manipulation des protocoles de chiffrement asymétrique

### 3.3.1 -  Rappels théoriques

Les 2 applications principales du chiffrement asymétriques sont : 
 - le chiffrement de message ou de fichier
 - Garantir l'authenticité de l'expéditeur

Le chiffrement : 
Cela marche assez simplement, l'expéditeur chiffre le message avec la clef public de son destinataire. Le destinataire pourra déchiffrer le message avec sa clef privée.

L'authenticiter : 
L'authenticier permet de garantir l'identité de notre expéditeur via une signature electronique. Cela permet donc l'intégriter, l'authenticité et la non répudiation. L'expéditeur va utilisé sa clef privé pour chiffrer un message que le destinataire pourra déchiffrer en utilisant la clef public de l'expéditeur, d'où le nom de "signature electronique".

### 3.3.2 - Préparation de la biclef

**Contexte : **
Utilisation de mon poste et de mon serveur distant.

**Génération sur poste :** 
Création des clefs sur mon poste :
`openssl genrsa -out key_poste_pv.pem 2048`
Extraction de la clef publique depuis la privée : 
`openssl rsa -in key_poste_pv.pem -outform PEM -pubout -out key_poste_pub.pem`

**Génération sur serveur : **
Création des clefs sur mon poste :
`openssl genrsa -out key_srv_pv.pem 2048`
Extraction de la clef publique depuis la privée : 
`openssl rsa -in key_srv_pv.pem -outform PEM -pubout -out key_srv_pub.pem`



| Environnement | PRIVE | PUBLIC |
| -------- | -------- | -------- |
| POSTE    | key_poste_pv.pem |  key_poste_pub.pem|
| SERVEUR  | key_srv_pv.pem | key_srv_pub.pem |

Afin de communiquer, on va devoir s'échanger nos clefs public, voila à quoi ressemble mon poste et celui de mon serveur : 

POSTE : 
![](https://i.imgur.com/D4Mgywh.png)

SERVEUR : 
![](https://i.imgur.com/s91lsSE.png)


### 3.3.3 - Chiffrement et déchiffrement de fichiers

Création du fichier : 
`echo "Salut c'est encore Adrien ahah" > secret_file.txt`

Chiffrement du fichier avec la clef public du serveur : 
`openssl rsautl -encrypt -inkey key_srv_pub.pem  -pubin -in secret_file.txt  -out secret_file.enc`

Envoie du fichier sur le serveur via scp.

Déchiffrement du fichier avec la clef privé du serveur : 
`openssl rsautl -decrypt -inkey key_srv_pv.pem -in secret_file.enc > top_secret.txt`

Et voila ce qu'on obtient : 

![](https://i.imgur.com/jJVFsgu.png)

### 3.3.4 - Signature électronique

Tout d'abord nous allons signer un fichier : 
Pour se faire il nous faut donc un fichier et notre clé privé.

`openssl dgst -sha1 -sign rsakeys/key_poste_pv.pem -out sha1.sign fichier1.txt`

Notre fichier est maintenant signé, nous allons donc envoyer à notre serveur  : 
 - Notre clef public
 - Notre fichier d'origine
 - Notre fichier sha1.sign

Voila la commande qu'il devra executer pour verifier l'authenticité de notre fichier : 
`openssl dgst -sha1 -verify key_poste_pub.pem -signature sha1.sign fichier1.txt`

Voila le résultat de la commande : 

![](https://i.imgur.com/05bf7Zh.png)

On voit bien que le fichier est vérifié et qu'il n'a donc pas été altéré :)