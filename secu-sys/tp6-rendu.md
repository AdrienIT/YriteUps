# TP6 - Keepass Avancé

## Préparation du serveur distant 

-> SSH
![](https://i.imgur.com/97SGTU9.png)

-> Keepass folder
![](https://i.imgur.com/1DYrpRA.png)

-> Keepass user
![](https://i.imgur.com/okr8wxc.png)

## Préparation du client : 

-> Installation de winfsp & sshfs
![](https://i.imgur.com/XmkrT8U.png)

###  Ajout du directory distant dans l'explorateur windows
-> Création du lecteur - local
[](https://i.imgur.com/nYGslBR.png)

-> Ajout du serveur - remote
![](https://i.imgur.com/sOrf7bl.png)

-> Mire d'authentification - local 
![](https://i.imgur.com/xv9iiCL.png)

-> Accès au dossier distant
![](https://i.imgur.com/PUuGnCB.png)

## Modification script robocopy pour gérer la copie de la base

```powershell
$dbsource = Read-Host "Please specify where the database is"

$dbname = Read-Host "Please specify which name the database have"

$testpath = $dbsource+$dbname

If((Test-Path -Path $dbsource) -and (Test-Path -Path $testpath -PathType Leaf))
{
    Write-Host "Establishing a new connection to remote server ..."
    net use Z: \\sshfs\keepass@<REDACTED>\opt\keepass /user:keepass "<REDACTED>"
    Write-Host "ROBOCOPY innnnggg"
    robocopy $dbsource $dbdest $dbname
    Write-Host "Disconnecting from server and deleting mounting point"
    net use /delete Z:
    Write-Host "DONE ...."

}else{
    Write-Host "The file or folder does not exist"
}

```

Preuve du fonctionnement : 
 - Resultat sur le client
 ![](https://i.imgur.com/Bhfx6sM.png)
 - Resultat sur le serveur
 ![](https://i.imgur.com/GYnex4E.png)

