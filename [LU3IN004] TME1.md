
# Publication

Publishing in StackEdit makes it simple for you to publish online your files. Once you're happy with a file, you can publish it to different hosting platforms like **Blogger**, **Dropbox**, **Gist**, **GitHub**, **Google Drive**, **WordPress** and **Zendesk**. With [Handlebars templates](http://handlebarsjs.com/), you have full control over what you export.

*__COMPTE-RENDU LU3IN004 n°1:__* *AGUINI Lydia (), GIANG Cécile (3530406)*

# ALMO TP n°1 - Simulateur MARS

## 1. Addition de deux valeurs stockées en mémoire

**Code de la fonction `sum`**

```nasm
.data
	var1: word 0x12
	var2: word 0x34
	var3: word 0  	#var3 contiendra le résultat

.text
.globl main
main:
	la $11, var1
	la $12, var2
	la $13, var3
	
	lw $21, 0($11)
	lw $22, 0($12)
	add $23, $22, $21
	sw $23, 0($13)
	
	li $2, 10
	syscall
```

*__Question:__* *A quoi correspond la première instruction du segment `text`?*

*__Question:__* *A quelles adresses sont rangées les trois variables `var1`, `var2` et `var3`?*

`var1` est rangé à l'adresse 0x1001000, `var2` en 0x1001004 et `var3` en 0x1001008.

*__Question:__* *Lancer l'exécution du programme en mode pas à pas, en analysant pour chaque instruction quels registres du processeur et quelles cases de la mémoire externe ont été modifiées.*

```nasm
.data
	var1: word 0x12		# valeur de var1: 0x00000012 à l'adresse mémoire 0x1001000
	var2: word 0x34		# valeur de var2: 0x00000034 à l'adresse mémoire 0x1001004
	var3: word 0		# valeur de var3: 0x00000000 à l'adress mémoire 0x1001008

.text
.globl main
main:
	la $11, var1		# $1 et $11 reçoivent 0x1001000 (adresse de var1)
	la $12, var2		# $12 reçoit 0x1001004 (adresse de var2)
	la $13, var3		# $13 reçoit 0x1001008 (adresse de var3)
	
	lw $21, 0($11)		# $21 reçoit 0x00000012 (valeur de var1, contenue à l'adresse stockée dans $11)
	lw $22, 0($12)		# $22 reçoit 0x00000034 (valeur de var2, contenue à l'adresse stockée dans $12)
	add $23, $22, $21	# $23 reçoit 0x00000046 (somme de var1 et var2, dont les valeurs sont stockees dans $21 et $22)
	sw $23, 0($13)		# on stocke dans l'adresse contenue (donc dans var3) dans $13 la valeur stockée dans $23
	
	li $2, 10			#exit(0)
	syscall
```

## 2. Somme des dix premiers entiers

**Code de la fonction `sum10`**

```nasm
.text
	li $5,0		#résultat
	li $2,0		#compteur de boucle
	li $6,11	#borne supérieure de la boucle

j  and 
for:
	addu $5,$5,$2
	addiu  $2,$2,1

and: bne $2,$6,for

li $2,10
syscall  
```

*__Question:__* *Vérifier le résultat obtenu.*
Après exécution du code, la valeur stockée dans `$5` est 0x00000037, ce qui donne 55 en décimal. Le résultat est alors bien celui que l'on attendait.


## 3. Conversion binaire - hexadécimal ASCII

*__Question:__* *Ecrire le programme assembleur correspondant au programme C donné.*

```nasm
.data
	table: .ascii  "0123456789ABCDEF"		# table des caractères ASCII des chiffres hexadécimaux
	string : .asciiz "0x________"			# chaîne de caractères résultat
	n:  .word 0x5432ABCD					# nombre à convertir

.text
	la $3,string							# $3 reçoit l'adresse mémoire de string
	addiu $3,$3,2							# PS: on se décale de 2 octets pour se placer à la première case de string à remplir 
	li $5,32								# $5 reçoit i = 32
	la $7,n									# $7 reçoit l'adresse mémoire de n
	lw $7,0($7)								# lecture en memoire de n et scotckage dans $7
	la $8,table								# $8 reçoit l'adresse mémoire de table
 
do:											
	addiu $5,$5,-4							# i = i - 4
 	srav   $6, $7,$5 						# n >> i
 	andi $6,$6,0x000f
 
	addu $9,$8,$6							#table+q
	lb $9,0($9)								# lecture en mémoire de table[q]
	sb $9,0($3)								# ecriture memoire de table[q] dans $3
	addiu $3,$3,1							# PS++
 bgtz  $5,do 
 
	#AFFICHAGE
	la $4,string 
	li $2,4
	syscall 

	#SORTIE DE PROGRAMME
	li $2,10
	syscall 
```
