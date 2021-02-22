*__COMPTE-RENDU LU3IN004 n°10:__* *AGUINI Lydia (3874863), GIANG Cécile (3530406)*

# ALMO TP n°10 - Plateforme multi-processeur

## 1. Différenciation des processeurs et partage de la mémoire

**_Question:_** *Ouvrez le fichier `main0.c` et complétez ce code pour que chaque processeur affiche un message différent dépendant du numéro de processeur.*

```nasm
tty_printf(" hello %d\n",procid());
```

**_Question:_** *Complétez le fichier `reset.s` en y ajoutant les instructions assembleur nécessaires à la réservation d'une pile séparée pour chaque processeur.*

```nasm
	 /* initializes stack pointer */
    mfc0    $10,    $15,    1
    andi    $10,    $10,    0x7         /* $10 <= proc_id */
    li      $27,   0x10000 		/* $27 <= 64 K */
    mult    $10,$27
    mflo    $26      			/* $26 <= proc_id*64K */
    addu    $26,$26,$27      		/* $26 <= (proc_id + 1)*64K */
    la      $29,    seg_stack_base      /* $29 <= seg_stack_base */
    addu    $29,$29,$26    		  /* $29 <= seg_stack_base + (proc_id + 1)*64K */
```
_On fixe à 64 Koctets la taille de la pile pour chaque processeur._


**_Question:_** *Vérifiez le bon fonctionnement en compilant et en exécutant successivement l'application logicielle sur le simulateur `simul_almo_generic`, en spécifiant 1, 2 ou 4 processeurs.*

**Avec 1 processeur :** 

L'affichage est correct :
```nasm
hello 0
``` 

**Avec 2 processeurs :**

Le fonctionnement est correct : deux terminaux s'ouvrent affichant `hello 0` et `hello 1`

**Avec 4 processeurs :**

Le fonctionnement est correct : quatre terminaux s'ouvrent et on a les affichages: `hello 0`, `hello 1`, `hello 2`, `hello 3`. 
		

## 2. Application parallèle multi-tâches coopératives

**_Question_** *Modifiez le fichier `Makefile` pour générer un exécutable `app.bin` à partir de `main1.c` et exécutez l'application sur une architecture mono-processeur.*

On rajoute au fichier `Makefile` le fichier `main1.c` dans les fichier sources SRC : SRC = main1.c

**_Question_** *Modifiez le fichier `main1.c` de façon à ce que le calcul puisse être exécuté sur 1 processeur, ou sur K processeurs (K compris entre 1 et 8). L'idée générale est la suivante : si on utilise deux processeurs, le processeur P[0] traite les blocs d'index (2n), et le processeur P[1] traite les blocs d'index (2n+1). Si on utilise 3 processeurs, le processeur P[0] traite les blocs d'index (3n), le processeur P[1] traite les blocs d'index (3n+1), et le processeur P[2] traite les blocs d'index (3n+2). Si on utilise K processeur le processeur P[k] traite les blocs d'index (K*n + k)* *sachant que k est compris entre 0 et (K-1).*

**Dans le fichier `main.c`:** 	

```nasm
for (line = 0; line < NLINES/nb_procs; ++line)
    	{
            time = proctime();
            tty_printf("processor %d processing line %d at time %d\n", my_index, line, time);
            filter(line*nb_procs + my_index);
    	}
```

`line < NLINES/nb_procs :` pour diviser les lignes en blocs traités chacun par un processeur 
    	
`filter(line*nb_procs + my_index) :` le processeur p[k] traite les blocs ( K*n + k ).

**_Exécutions:_**

**Cas à 1 processeur :**
Il se termine après 9042540 cycles.
	
**Cas à 2 processeurs :** 
Le processeur 0 termine sa tâche après 4479394 cycles. Le processeur 1 termine après 4505561 cycles.
	
**Cas à 4 processeurs :** 
Le processeur 0 termine après 2216809 cycles. Le processeur 1 termine après 2211077 cycles. Le processeur 2 termine après 2211129 cycles. Le processeur 3 termine après 2244130 cycles.
	
**Cas à 8 processeurs :** 
Le processeur 0  termine après 1144038 cycles.
Le processeur 1 termine après 1139582 cycles.
Le processeur 2 termine après 1137536  cycles.
Le processeur 3 termine après 1138488 cycles.
Le processeur 4  termine après 1139327  cycles.
Le processeur 5 termine après 1136835  cycles.
Le processeur 6 termine après 1137386  cycles.
Le processeur 7 termine après 1177768  cycles.


**_Question:_** *Tracez à l'aide de `gnuplot` la courbe représentant en abscisse le nombre de processeurs K, et en ordonnée l'accélération speedup[K]. Quelles conclusions peut-on tirer de cette courbe ?*

On constate que plus le temps est partagé entre les processeurs de la plateforme, plus on va vite dans les exécutions.

La courbe représentant en abscisse le nombre de processeurs K, et en ordonnée l'accélération speedup[K]  est décroît exponentiellement. 
Plus le nombre de processeurs augmente, plus vite la valeur speedup[k] baisse. En atteignant 8 processeurs, l'accélération se stabilise et prend une valeur presque constante; au-delà de 8 processeurs, on n'a plus de gain sur les exécutions. Au contraire cela peut même nuire aux performances du système.
