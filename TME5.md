*__COMPTE-RENDU LU3IN004 n°5:__* *AGUINI Lydia (3874863), GIANG Cécile (3530406)*

# ALMO TP n°5 - Mémoires Cache

## 1. Calcul du taux de MISS dans le cache d'instructions

*__Question:__* *Ouvrez le fichier `main.c` et expliquez ce que fait la fonction `main()` ?*

La fonction main initialise un tableau de 15 cases à 0 puis puis ajoute 1000 fois la valeur `i` à la case numéro `i` du vecteur, avant de les afficher en sortie.

*__Question:__* *Lancez l'exécution du `Makefile`, puis examinez le code assembleur correspondant à l'application logicielle (`app.bin.txt`). Déterminez les adresses de début et de fin de la boucle de calcul.*
  
La 2ème boucle `while` commence à l'adresse `0x401334` et se termine en `0x4013fc`.

*__Question:__* *Combien d'instructions sont exécutées à chaque itération de cette boucle ?*

51 instructions sont exécutées à chaque itération de boucle.

*__Question:__* *En éditant le fichier `app.bin.txt`, déterminez, pour chaque instruction de la boucle de calcul, à quelle ligne de cache cette instruction appartient. On regroupera les 51 instructions du corps de boucle par groupe de 4 (puisqu'une ligne de cache contient 4 instructions).*

__*1ère ligne :*__

```nasm
401330:  00000000 nop  0000 0000 0100 0000 0001 0011 0011 0100  c

401334:  8fc20018 lw  v0,24(s8)

401338:  afc20018 sw  v0,24(s8)

40133c:  8fc2001c lw  v0,28(s8)
```

__*2ème ligne :*__ `index = 4`

```nasm
401340:  24420001 addiu  v0,v0,1  0000 0000 0100 0000 0001 0011 0100 0100  index = 4

401344:  afc2001c sw  v0,28(s8)

401348:  8fc20020 lw  v0,32(s8)

40134c:  24420002 addiu  v0,v0,2
```

__*3ème ligne :*__ `index = 5`

```nasm
401350:  afc20020 sw  v0,32(s8)

401354:  8fc20024 lw  v0,36(s8)  0000 0000 0100 0000 0001 0011 0101 0100  index = 5

401358:  24420003 addiu  v0,v0,3

40135c:  afc20024 sw  v0,36(s8)
```

__*4ème ligne :*__ `index = 6`

```nasm
401360:  8fc20028 lw  v0,40(s8)

401364:  24420004 addiu  v0,v0,4  0000 0000 0100 0000 0001 0011 0110 0100  index = 6

401368:  afc20028 sw  v0,40(s8)

40136c:  8fc2002c lw  v0,44(s8)
```

__*5ème ligne :*__ `index = 7`

```nasm
401370:  24420005 addiu  v0,v0,5

401374:  afc2002c sw  v0,44(s8)  0000 0000 0100 0000 0001 0011 0111 0100  index = 7

401378:  8fc20030 lw  v0,48(s8)

40137c:  24420006 addiu  v0,v0,6
```

__*6ème ligne :*__ `index = 0`

```nasm
401380:  afc20030 sw  v0,48(s8)

401384:  8fc20034 lw  v0,52(s8)  0000 0000 0100 0000 0001 0011 1000 1000  index = 0

401388:  24420007 addiu  v0,v0,7

40138c:  afc20034 sw  v0,52(s8)
```

__*7ème ligne :*__ `index = 1`

```nasm
401390:  8fc20038 lle clw  v0,56(s8)

401394:  24420008 addiu  v0,v0,8  0000 0000 0100 0000 0001 0011 1001 0100  index = 1

401398:  afc20038 sw  v0,56(s8)

40139c:  8fc2003c lw  v0,60(s8)
```

__*8ème ligne :*__ `index = 2`

```nasm
4013a0:  24420009 addiu  v0,v0,9

4013a4:  afc2003c sw  v0,60(s8)  0000 0000 0100 0000 0001 0011 1010 0100  index = 2

4013a8:  8fc20040 lw  v0,64(s8)

4013ac:  2442000a addiu  v0,v0,10
```

__*9ème ligne :*__ `index = 3`

```nasm
4013b0:  afc20040 sw  v0,64(s8)

4013b4:  8fc20044 lw  v0,68(s8)  0000 0000 0100 0000 0001 0011 1011 0100  index = 3

4013b8:  2442000b addiu  v0,v0,11

4013bc:  afc20044 sw  v0,68(s8)
```

__*10ème ligne :*__ `index = 4`

```nasm
4013c0:  8fc20048 lw  v0,72(s8)

4013c4:  2442000c addiu  v0,v0,12  0000 0000 0100 0000 0001 0011 1100 0100  index = 4

4013c8:  afc20048 sw  v0,72(s8)

4013cc:  8fc2004c lw  v0,76(s8)
```

__*11ème ligne :*__ `index = 5`

```nasm
4013d0:  2442000d addiu  v0,v0,13

4013d4:  afc2004c sw  v0,76(s8)  0000 0000 0100 0000 0001 0011 1101 0100  index = 5

4013d8:  8fc20050 lw  v0,80(s8)

4013dc:  2442000e addiu  v0,v0,14
```

__*12ème ligne :*__ `index = 6`

```nasm
4013e0:  afc20050 sw  v0,80(s8)

4013e4:  8fc20014 lw  v0,20(s8)  0000 0000 0100 0000 0001 0011 1110 0100  index = 6

4013e8:  24420001 addiu  v0,v0,1

4013ec:  afc20014 sw  v0,20(s8)
```

__*13ème ligne :*__ `index = 7`

```nasm
4013f0:  8fc20014 lw  v0,20(s8)

4013f4:  2c4203e8 sltiu  v0,v0,1000  0000 0000 0100 0000 0001 0011 1111 0100  index = 7

4013f8:  1440ffce bnez  v0,401334 <main+0x58>

4013fc:  00000000 nop
```

*__Question:__* *Combien d'instructions sont exécutées à chaque itération de cette boucle ? Toutes les instructions de la boucle de calcul peuvent-elles être simultanément stockées dans le cache ?*

À chaque itération de boucle, 51 instructions sont exécutées, ce qui revient à  `51 * 4 octets = 204 octets` or mon cache à une capacité de `128 octets`. Ces instructions ne peuvent être simultanément en mémoire.  

*__Question:__* *Que pouvez-vous en conclure ?*

On en conclut qu'il y aura des `MISS` durant l'exécution de la boucle.

*__Question:__* *Évaluez le nombre de `MISS instruction` lors de l'exécution de la première itération ? Lors de la deuxième itération ? En déduire une valeur estimée du **taux de MISS** moyen après 1000 itérations. ?*

Sachant que lors de la lecture en mémoire, on va charger toute une ligne de cache:

1) **1ère itération :** 13 `MISS` (certaines cases seront écrasées)

2) **2ème itération:** 10 `MISS` (les cases `0`, `1` et `2` ne provoqueront pas de `MISS` mais il y a écrasement des cases `3`, `4`, `5`, `6`, `7`)

3) **3ème itération :** 10 `MISS` _(idem)_

**Calcul du taux de `MISS` :**

`TAUX DE MISS = Nombre de MISS / Nombre total d'accès mémoire = ((10*999)+13)/(51*1000) = 20%`

Car on a 10 `MISS` (999 fois), 13 `MISS` à la première itération et 51 instructions(1000).  

## 2. Analyse de trace

*__Question:__* *À quel cycle est chargée dans le cache d'instructions la première instruction de la fonction `main()` ?*

Au cycle 57.

*__Question:__* *À quel cycle est chargée la première ligne de cache contenant des instructions de la boucle de calcul ?*

Au cycle 380.

*__Question:__* *À quel cycle cette première ligne est-elle évincée par le chargement d'une autre ligne de cache ?*

Au cycle 553.

*__Question:__* *À quel cycle cette première ligne est-elle rechargée pour exécuter la deuxième itération de la boucle ?*

Au cycle 641.

*__Question:__* *A quel cycle est-elle rechargée pour exécuter la troisième itération?*

Au cycle 815.

*__Question:__* *Quelle est la durée (en nombre de cycles) de la première itération?*

La durée est de 641 - 380 = 261 cycles.

*__Question:__* *Quelle est la durée des itérations suivantes?*

La durée est de 815 - 641 = 174 cycles.


## 3. Mesure du taux de MISS

*__Question:__* *Comment expliquez-vous l'évolution du taux de `MISS` au cours du temps ?*

À partir d'une certaine valeur de temps, le graphe se stabilise à la valeur `y = 0.2` (qui confirme bien notre approximation d'un taux de `MISS` à 20%), donc avant cela le taux de `MISS` total augmentait à chaque tour de boucle.

## 4. Optimisation du code pour minimiser le taux de MISS

*__Question:__* *Comment expliquez-vous l'évolution du taux de `MISS` pour cette nouvelle version de l'application ?*

Apres réécriture de la fonction `main()` avec une boucle, la longueur du code `asm` diminue: la boucle n'a alors plus que 16 instructions, ce qui tient dans le cache. Le nombre de `MISS` diminue cette fois ci après remplissage du cache jusqu'à une valeur presque nulle.
