# TP 5 - Keepass

## Découverte de l'outil

Les algorithmes de chiffrements sont AES et TwoFish.

Il est préférable d'utiliser va version 2.41 plutôt que la 1.36 car la version 1.36 contient des failles au niveau du chiffrement.

## Création du compte admin 
![](https://i.imgur.com/OAYZ9Tf.png)

## Utilisation du remplissage automatique

### Email

![](https://i.imgur.com/p4gphzn.gif)

### Cmd

![](https://i.imgur.com/8xLJXkp.png)


## Changement du mot de passe de la database

![](https://i.imgur.com/XZumhEs.png)

## Fonctions triggers

Dans Keepass, les fonctions de triggers servent à automatisé des choses, tel que la sauvegarde automatique de la base lors de la fermeture du programme.

### Création de notre trigger

### Script robocopy

```ps1
$dbsource = Read-Host "Please specify where the database is"

$dbname = Read-Host "Please specify which name the database have"

$dbdest = Read-Host "Please specify the place you want to save tour file"

$testpath = $dbsource + $dbname

If((Test-Path -Path $dbsource) -and (Test-Path -Path $dbdest) -and (Test-Path -Path $testpath -PathType Leaf))
{
    Write-Host "ROBOCOPY innnnggg"
    robocopy $dbsource $dbdest $dbname
}else{
    Write-Host "The file or folder does not exist"
}

```