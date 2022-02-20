# Pentest C3 - Certificate Pinning

## Contexte

Ici, nous avons une nouvelle application avec quelques check à bypasse.
Notre intervenant nous a demandé de bypass okhttp et trustkit, c'est ce que l'on va voir aujourd'hui

### okhttp

Pour cette fonction c'est assez simple.
EN explorer un peu les fichier on se rends comte qu'un "CerificatePinner.smali" existe et qu'il gére la conition pour okhttp.

Avec jadx on observe que la condition qui va nous intéresser est celle situé ligne 403.

Il va juste falloir inverser la condition et passer de if-eqz à if-nez.

En recompilant et en poussant l'app sur notre emulateur on se rend compte que le check passe : 
![](https://i.imgur.com/n5T0LuZ.png)

### TrustKit

Enfin pour trustkit, c'est la meme chose, on vient chercher la fonction puis la condition a inverser pour passer le check.

pour se faire je me suis penché sur le fichier PinningTrustManager.

J'ai remarqué une condition ligne 147 que je pensais inverser. C'est ce que j'ai fais dans mon fichier smali mais ça ne passait pas.

J'ai tenté d'inverser d'autres fonctions mais cela ne marchait pas non  plus

Je n'ai donc pas réussi cette 2eme étape.