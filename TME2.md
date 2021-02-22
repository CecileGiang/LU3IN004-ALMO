*__COMPTE-RENDU LU3IN004 n°2:__* *AGUINI Lydia (3874863), GIANG Cécile (3530406)*

# ALMO TP n°2 - Fonctions imbriquées et récursives

## 1. Moyenne de N entiers stockés dans un tableau

**Segment .data**

```nasm
.data
	tab: .word 23, 7, 12, 513, -1
```

**Code de la fonction `main`**

```nasm
.text

.globl main

main:
	# PROLOGUE: nr = 1 ($31), nv = 1 (variable `x`), na = 1

	addiu $29, $29, -12			# -4*3 = -12
	sw $31, 8($29)
	la $4, tab					# on stocke en $4 l'adresse mémoire de tab
	sw $4, 4($29)
	jal arimean					# appel à la fonction arimean
	add $20, $0, $2				# on stocke le résultat de l'appel dans $20
	
	#Affichage du résultat x
	add $4,$2,$0
	li $2, 1
	syscall
	
	# SORTIE DE PROGRAMME
	li $2, 10
	syscall	
```

**Code de la fonction `arimean`**

```nasm
arimean:
	
	# PROLOGUE: nr = 1 ($31),  nv = 2 (x = $20 et n = $21),  na = 1
	addiu $29, $29, -16
	sw $31, 8($29)

	# CORPS
	jal sizetab			# appel à sizetab
	add $21, $2, $0		# stockage du résultat de l'appel n dans $21
	lw $4, 20($29)		# relecture de tab
	jal sumtab			# appel à sumtab
	add $20, $2, $0		# stockage du résultat de l'appel x dans $20

	# Division de x par n
	div $20,$21
	mflo $2

	# EPILOGUE
	lw $31, 8($29)
	addiu $29, $29, 16
	jr $31
```

**Code de la fonction `sizetab`**

```nasm
sizetab:

	# PROLOGUE: nr = 0,  nv = 1 (index = $22),  na = 0
	addiu $29, $29, -4

	# CORPS
	
	li $22, 0				# $22: index = 0
	
	j cond1:
while1:
	addiu $22, $22, 1		# index += 1
cond1:
	sll $23, $22, 2			# $23: index*4
	add $23, $4, $23		# $23: emplacement courant dans tab: tab + index*4
	lw $23, 0($23)			# stockage dans $23 de tab[index]
	bgez $23, while1

	add $2, $0, $22

	# EPILOGUE
	addiu $29, $29, 4
	jr $31
```

**Code de la fonction `sumtab`**

```nasm
sumtab:

	# PROLOGUE: nr = 0,  nv = 2 (index = $22 et accu = $23), na = 0
	addiu $29, $29, -8

	# CORPS

	li $22, 0			# $22: index = 0
	li $23, 0			# $23: accu = 0

	jcond2
while:
	add $23,$23,$24		#	accu = accu + t[index]  
	addiu $22,$22,1		# index += 1
cond2:
	sll $24,$22,2		# $24: index*4
	add $24,$4,$24		# $24: emplacement courant dans tab: tab + index*4
	lw $24,0($24)		# $24: t[index] 
	bgez ,$24,while2
	
	add $2,$0,$23

	# EPILOGUE
	addiu $29,$29,8
	jr $31
```

L'exécution de ce programme nous affiche un résultat de 138, ce qui correspond bien à ce l'on attendait (nous avons fait une division entière, le vrai résultat est de 138,75).


## 2. Fonctions récursives: calcul de la factorielle

**Code de la fonction `main`**

```nasm
.data
	nombre: .word 0x04			# nombre dont on veut calculer la factorielle

.text
.globl main

main:
	# PROLOGUE: nr = 1 ($31),  nv = 0,  na = 1
	
	addiu $29,$29,-8
	sw $31,0($29)
	
	la $4,nombre		# $4: adresse en mémoire de la variable nombre
	lw $4, 0($4)		# $4: nombre
	
	# Note: pour n = 1000, le résultat est 0 (dépassement de capacité),et pour n < 0, on a une boucle infinie

	jal fact			# appel à la fonction factorielle
	
	# AFFICHAGE DU RÉSULTAT
	add $4,$2,$0
	li $2,1
	syscall 

	# SORTIE DE PROGRAMME
	addiu $29,$29,8
	li $2,10
	syscall 
```

**Code de la fonction `fact`**

```nasm
fact:
	# PROLOGUE: nr = 2,  nv = 0,  na = 1 (récursion)
	addiu $29,$29,-12
	sw $31,0($29)
	sw $16,4($29)			
	
	# CORPS
	ori $16,$4,0 
	li $2,1					# $2 mis à 0 pour traiter le cas n == 0

	beq $4,$0,epilogue		# cas n == 0: on renvoie $2 = 1

	# cas n > 0: Récursion
	addi $4,$4,-1			# n -= 1
	jal fact

	mult $2,$16				# mise à jour du résultat final de la factorielle
	mflo $2

epilogue:
	lw $31,0($29)
	lw $16,4($29)
	addiu $29,$29,12
	jr $31
```

Nous avons testé ce programme. Il donne bien les résultats attendus.
