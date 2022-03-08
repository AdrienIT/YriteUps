# TP 3 : Sécurisation des sessions distantes avec openssh

## Contexte : 

Dans notre cas, on utlisera un serveur ssh basé sur ubuntu 21.10 impish.

### 3.1 Rappels théoriques : 

Les 2 applications principales du chiffrement asymétriques sont :

- le chiffrement de message ou de fichier
- Garantir l'authenticité de l'expéditeur

Le chiffrement : Cela marche assez simplement, l'expéditeur chiffre le message avec la clef public de son destinataire. Le destinataire pourra déchiffrer le message avec sa clef privée.

L'authenticiter : L'authenticier permet de garantir l'identité de notre expéditeur via une signature electronique. Cela permet donc l'intégriter, l'authenticité et la non répudiation. L'expéditeur va utilisé sa clef privé pour chiffrer un message que le destinataire pourra déchiffrer en utilisant la clef public de l'expéditeur, d'où le nom de "signature electronique".

### 3.2 Préparation du serveur

#### 3.2.2 Première connexion : 

Remove root login : 

Dans /etc/ssh/sshd_config on passe la la ligne de PermitRootLogin de on à off

![](https://i.imgur.com/jGrKaX5.png)

SSH2 est bien utilisé lors de la connection SSH
![](https://i.imgur.com/jfjY1WJ.png)

Pour augmenter la sécurité de notre serveur : 
 - Garder les packets à jour
 - Utiliser le fichier authorized_keys
 - Enlever le login via le user root
 - Faire du filtrage IP



#### 3.2.3 Deuxième connexion : 

En répondant "no" lors de la première connexion, voilà le message que l'on a  : 

![](https://i.imgur.com/QFtUs3t.png)


Le but ici, c'est qu'on se connecte à la machine voulu et rien qu'à la machine voulue. On ne veut pas s'authentifier sur une machine pirate ou se faire prendre par un Man-In-The-Middle.

Le fingerprint sha256 de notre serveur : 
`debug1: Server host key: ecdsa-sha2-nistp256 SHA256:/dtxU5AxBSiAsdIRuYxeMgGv3lFiOr98dzmihMTFzCc
`
Pour récupérer les fingerprint d'une machine : 
![](https://i.imgur.com/LupuZeR.png)

Une autre option aurait était d'allai voir dans le fichier ~/.ssh/known_hosts pour voir si notre machine connaissait deja ce host.

Si on réponds oui à la question, les différents fingerprint vont s'ajouter à au fichier ~/.ssh/known_hosts.
![](https://i.imgur.com/o5JJGmM.png)

### 3.3 Préparation du client UNIX/BSD : 

D'après les recommendationsde l'ANSSI, pour un host de 3 ans, voilà ce qu'il faut faire concernant les tailles : 

![](https://i.imgur.com/msqnA2q.png)

- Une taille de 2048 bits pour RSA
- Une taille de 256 bits pour ECDSA

Et concernant les algo, voilà ce que l'on retrouve : ![](https://i.imgur.com/4DbH1ds.png)

- De l'AES128 avec de la randomisation et une clef privé
- PKCS12 pour l'importation et donc l'utilisation sans clef privée

Génération des clefs : 
Clef privée : 

![](https://i.imgur.com/u3Plqn7.png)


Extraction de la clef public via la clef privée : 

![](https://i.imgur.com/raf0Rdb.png)

Au niveau des droits d'accès de ma clef privé, j'ai mis du chmod 600.
Ce qui signifie que seul l'utilisateur peut : modifier et lire mais pas executer, les autres n'ont aucuns droits.

L'option "StrictModes" rejete l'utilisation de l'authentification par clé publique et privée si les fichiers requis ne sont pas correctement protégés.

Maintenant, on vient copier notre clef public dans /home/bob/.ssh/authorized_keys Ou sinon passe par le binaire ssh-copy-id :) 
![](https://i.imgur.com/BC7lN67.png)

Et la connection marche :o 