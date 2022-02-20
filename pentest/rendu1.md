# Pentest B3 - Reverse Android

## Rootbeer

### Rappel du contexte

Lors de ces 8 premières heures de pentest nous avons eu à "pentester" et à reverser une app android nommé "Rootbeer", le but étant de passer outre les vérification de sécurité mise en place par le dev l'appli.

Pour se faire nous avons eu besoin de quelques outils : 
 - GenyMotion, pour l'émulation de notre smartphone
 - Jadx, pour regarder le code de notre app
 - apktool , jarsigner et zipaligner, Rebuild de l'app après nos modifications

### Enumération des fonctions

Voici à quoi ressemble l'app au lancement : 

![](https://i.imgur.com/IYWW5Lh.png)


On obseve donc que l'on a 12 fonctions.
On a deja des fonctions en verte, il nous reste plus que 6 fonctions à reverse qui sont : 
 - TestKeys
 - BusyboxBinary
 - SuBinary
 - 2nd SuBinary
 - Dangerours props
 - Root via native check


### Fonction 1 : testkeys

On vient ouvrir notre app avec jad et on vient trouver la fonction qui nous intéresse avec un grep.

![](https://i.imgur.com/uGB6Q2o.png)

On va donc ouvrir notre code smali avec sublime text pour voir la fonction ligne 1161

![](https://i.imgur.com/obTKPzF.png)

Grace au site de pallergabor, on va vite comprendre de quoi il s'agit.

Le but va etre de retourner une valeur false lorsqu'on rentre dans la condition qui nous detecte.

Pour se faire on va venir modifier la valeur de 0x0 à la ligne 1183.
On va passer de 0x1 à 0x0 (soit passer de true à false)

Apres avoir rebuild l'app avec apktool, jarzigner et zipalign,
Voila ce que l'on observe : 


![](https://i.imgur.com/9rawpAk.png)

On a donc bien bypassé la verification, car on rentre dans la condition qui nous detecte, mais on lui dit de renvoyer false, on est vert !!!

### Fonction 2 : BusyboxBinary

Pareil avec un grep on vient chercher la ligne (ou un ctrl F)

voila ce que l'on observe : 

![](https://i.imgur.com/LutQHSJ.png)

Grace au site cité plus haut, on remarque que si l'on change move-result v0 à const/4 v0,  0x0. On bypass simplement la verification.

En effet puisque move-result renvoie la valeur d'une ancienne method dans un v0. Cela veut que l'on peut redefinir v0. on passe donc de true à false cette derneière et le tour est joué.

![](https://i.imgur.com/wvRZ2Kj.png)

### Fonction 3 : SU Binary

Pour cette fonction, voila ce qu'on a : 

![](https://i.imgur.com/xLcdPLq.png)

Comme au dessus, on va changer move-result à const/4 v0, ce qui va nous permettre de bypass la verification : 


![](https://i.imgur.com/cvCwGum.png)



### Fonction 4 : 2nd SU binary (CheckSuExists)

Une fonction un peu plus longue, je vous met le code ici : 

```smali
.method public checkSuExists()Z
    .locals 6

    const/4 v0, 0x0

    const/4 v1, 0x0


    .line 399
    :try_start_0
    invoke-static {}, Ljava/lang/Runtime;->getRuntime()Ljava/lang/Runtime;

    move-result-object v2

    const/4 v3, 0x2

    new-array v3, v3, [Ljava/lang/String;

    const-string v4, "which"

    aput-object v4, v3, v0

    const-string v4, "su"

    const/4 v5, 0x1

    aput-object v4, v3, v5

    invoke-virtual {v2, v3}, Ljava/lang/Runtime;->exec([Ljava/lang/String;)Ljava/lang/Process;

    move-result-object v1

    .line 400
    new-instance v2, Ljava/io/BufferedReader;

    new-instance v3, Ljava/io/InputStreamReader;

    invoke-virtual {v1}, Ljava/lang/Process;->getInputStream()Ljava/io/InputStream;

    move-result-object v4

    invoke-direct {v3, v4}, Ljava/io/InputStreamReader;-><init>(Ljava/io/InputStream;)V

    invoke-direct {v2, v3}, Ljava/io/BufferedReader;-><init>(Ljava/io/Reader;)V

    .line 401
    invoke-virtual {v2}, Ljava/io/BufferedReader;->readLine()Ljava/lang/String;

    move-result-object v2
    :try_end_0
    .catchall {:try_start_0 .. :try_end_0} :catchall_0

    if-eqz v2, :cond_0

    const/4 v0, 0x1

    :cond_0
    if-eqz v1, :cond_1

    .line 405
    invoke-virtual {v1}, Ljava/lang/Process;->destroy()V

    :cond_1
    return v0

    :catchall_0
    nop

    if-eqz v1, :cond_2

    invoke-virtual {v1}, Ljava/lang/Process;->destroy()V

    :cond_2
    return v0
.end method
```

Bon ici j'ai tenté quelque chose que je voulais faire depuis le début. QU'est ce qu'il se passe si je déclare une variable qui vaut False et que je la retourne directement après sa déclaration ?


Voila ce que j'ajoute au début : 

```smali
    const/4 v0, 0x0

    return v0
```

Et après avoir push sur notre android, on se rend compte que ça marche : 

![](https://i.imgur.com/IgPJfFr.png)


### Fonction 5 : Check For dangerous props

On aurait pu continuer comme la méthode d'au dessus, mais c'est trop simple.

Voila ce qu'on a : 

![](https://i.imgur.com/ozc8ttm.png)

Ici on remarque la déclaration d'une variable à True, qui sera retrounée plus tard dans le code. Et si  on changeant juste cette variable de True à False ?

On passe de 0x0 à 0x1, regardons le résultat : 

![](https://i.imgur.com/MMsgFxk.png)

Cool on bypass !

### Fonction 6 : CheckForRootNative

Voilà ce qu'on a  : 

```smali
.method public checkForRootNative()Z
    .locals 7

    .line 444
    invoke-virtual {p0}, Lcom/scottyab/rootbeer/RootBeer;->canLoadNativeLibrary()Z

    move-result v0

    const/4 v1, 0x0

    if-nez v0, :cond_0

    const-string v0, "We could not load the native library to test for root"

    .line 445
    invoke-static {v0}, Lcom/scottyab/rootbeer/util/QLog;->e(Ljava/lang/Object;)V

    return v1

    .line 449
    :cond_0
    invoke-static {}, Lcom/scottyab/rootbeer/Const;->getPaths()[Ljava/lang/String;

    move-result-object v0

    .line 451
    array-length v2, v0

    new-array v3, v2, [Ljava/lang/String;

    const/4 v4, 0x0

    :goto_0
    if-ge v4, v2, :cond_1

    .line 453
    new-instance v5, Ljava/lang/StringBuilder;

    invoke-direct {v5}, Ljava/lang/StringBuilder;-><init>()V

    aget-object v6, v0, v4

    invoke-virtual {v5, v6}, Ljava/lang/StringBuilder;->append(Ljava/lang/String;)Ljava/lang/StringBuilder;

    move-result-object v5

    const-string v6, "su"

    invoke-virtual {v5, v6}, Ljava/lang/StringBuilder;->append(Ljava/lang/String;)Ljava/lang/StringBuilder;

    move-result-object v5

    invoke-virtual {v5}, Ljava/lang/StringBuilder;->toString()Ljava/lang/String;

    move-result-object v5

    aput-object v5, v3, v4

    add-int/lit8 v4, v4, 0x1

    goto :goto_0

    .line 456
    :cond_1
    new-instance v0, Lcom/scottyab/rootbeer/RootBeerNative;

    invoke-direct {v0}, Lcom/scottyab/rootbeer/RootBeerNative;-><init>()V

    .line 458
    :try_start_0
    iget-boolean v2, p0, Lcom/scottyab/rootbeer/RootBeer;->loggingEnabled:Z

    invoke-virtual {v0, v2}, Lcom/scottyab/rootbeer/RootBeerNative;->setLogDebugMessages(Z)I

    .line 459
    invoke-virtual {v0, v3}, Lcom/scottyab/rootbeer/RootBeerNative;->checkForRoot([Ljava/lang/Object;)I

    move-result v0
    :try_end_0
    .catch Ljava/lang/UnsatisfiedLinkError; {:try_start_0 .. :try_end_0} :catch_0

    if-lez v0, :cond_2

    const/4 v1, 0x1

    :catch_0
    :cond_2
    return v1
.end method
```

On remarque tout de suite 2 variables et 2 returns

On décide donc de changer le dernier return et de le passer de true à false.

Voila ce qu'on obtient : 

![](https://i.imgur.com/gqKFo3v.png)

On bypass la vérification


## Conclusion

à la fin cela doit ressembler à ça : 
![](https://i.imgur.com/JbaiUwc.png)

Encore une fois, super TP, je n'avais jamais fait de reverse ANdroid et de bypasse de fonction.
On aurait pu croire cela très complexe au premier abord mais finalement en s'y penchant, ce n'est que de l'algorithmie simple ^^ .