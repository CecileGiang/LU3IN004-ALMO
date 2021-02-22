*__COMPTE-RENDU LU3IN004 n°3:__* *AGUINI Lydia (3874863), GIANG Cécile (3530406)*

# ALMO TP n°3 - Génération de code avec GNU GCC : exécution sous MARS

## 1. Exécution du programme de tri écrit directement en assembleur

## 2. Compilation d'un fichier C : exemple

### 2.1 Test simple

*__Question:__* *Commencer par éditer un petit programme C, sans appel de fonction, dans le fichier* `exemple.c`

```c
int main(void){
	int a, b;
	a = 24;
	b = 16;
	while (a != b) {
		if (a > b) a = a - b;
		else b = b - a;
	}
	return 0;
}
```

*__Question:__* *Que fait ce programme ?*

Ce programme permet de calculer le PGCD de deux entiers a et b positifs.

*__Question:__* *En éditant le fichier* `exemple.s`, *déterminer les noms des étiquettes* (`'labels'`) *intervenant dans ce programme. En déduire comment GCC fait pour générer les noms des étiquettes.*

Les labels intervenant dans ce programme sont: `main`,  `$L2`, `$L3` et `$L4`. On en déduit que pour générer les noms des étiquettes, GCC reprend le nom des fonctions contenues dans le fichier C, et il nomme le reste des étiquettes sous le format `$L` suivi d'un entier (entiers consécutifs à partir de 2).

*__Question:__* *Déterminer à quelles adresses sont stockées en mémoire les valeurs des deux variables locales* `a` *et* `b`.

```nasm
	li	$2,24			# 0x18
	sw	$2,0($sp)
	li	$2,16			# 0x10
	sw	$2,4($sp)
```
D'après ce morceau de code extrait de `exemple.s`, les variables locales `a` et `b` sont stockées dans la pile, respectivement à `0($29)` et à `4($29)`.

*__Question:__* *Identifier le code permettant d'initialiser la variable* `a` *avec la valeur* `24`. *Même question pour l'initialisation de* `b`.

Les variables `a` et `b` sont initialisées par les instructions suivantes:

```nasm
	# Initialisation de a
	li	$2,24			# 0x18
	sw	$2,0($sp)

	# Initialisation de b
	li	$2,16			# 0x10
	sw	$2,4($sp)
```
*__Question:__* *Déterminer comment le compilateur génère le code assembleur MIPS32 correspondant à l'instruction* `while`/`if`. *Quelles sont les étiquettes qui interviennent dans ce code ? En déduire la manière générale utilisée par GCC pour générer le code assembleur MIPS32 équivalent à l'instruction C* `while`/`if`.

D'après le code assembleur du fichier `exemple.s`, nous pouvons déduire comment le compilateur a généré notre boucle `while` :

* `$L2` est le label qui gère la boucle `while` dans le cas où `a != b` (boucle principale)
* `$L3` est le label qui gère le cas où la condition de boucle n'est pas respectée, et que celle du `if` est respectée
* `$L4` gère le cas où la condition du `while` est respectée mais pas celle du `if`

D'une manière générale:

* Le compilateur crée un label au nom de la fonction (ici `main`)
* Dans le cas où il existe dans le programme une structure de contrôle (`while`, `if`...), il crée deux labels, le premier traitant les cas où la condition est respectée, la deuxième traitant le cas où elle ne l'est pas
* Si ces structures de contrôle sont imbriquées, il crée deux labels comme décrits au point précédent pour la structure principale (la structure la plus extérieure), puis pour chaque nouvelle structure imbriquée dans celle-ci, le cas où la condition est respectée est traitée dans le label courant, et un nouveau label est créé pour traiter le cas échéant
* Les labels sont créés et nommés dans l'ordre d'apparition de la structure de contrôle correspondante. Ils sont tous sous la forme `$L` suivi d'un entier, qui indique l'ordre dans lequel ils sont créés (à partir de 2)

*__Question:__* *Exécuter le programme en pas à pas et vérifier que la valeur calculée par ce programme est correcte.*

Nous avons vérifié le contenu des registres, les valeurs obtenues sont correctes.


## 3. Compilation du fichier tri.c et utilisation de MARS

*__Question:__* *Où est stocké le tableau `tab` en mémoire ? Exécuter le programme en mode pas à pas pour comprendre comment ce tableau est initialisé.*

Dans `tri.s`, le tableau `tab` est initialisée dans la zone `.data` (il était automatiquement initialisé dans la zone `.text` par `MARS` mais nous avons dû le déplacer dans `.data` pour ne pas produire d'erreur) par les instructions suivantes:
 
```nasm
$LC0:
	.word	3
	.word	33
	.word	49
	.word	4
	.word	23
```
Il est ainsi référencé par le label `$LC0`, qui contient l'adresse de la première instruction qui la suit (donc l'adresse de `.word 3`, le premier élément du tableau).
