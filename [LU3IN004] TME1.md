---


---

<p><em><strong>COMPTE-RENDU LU3IN004 n°1:</strong></em> <em>AGUINI Lydia (), GIANG Cécile (3530406)</em></p>
<h1 id="almo-tp-n°1---simulateur-mars">ALMO TP n°1 - Simulateur MARS</h1>
<h2 id="addition-de-deux-valeurs-stockées-en-mémoire">1. Addition de deux valeurs stockées en mémoire</h2>
<p><strong>Code de la fonction <code>sum</code></strong></p>
<pre class=" language-nasm"><code class="prism  language-nasm">.data
	<span class="token label function">var1:</span> word <span class="token number">0x12</span>
	<span class="token label function">var2:</span> word <span class="token number">0x34</span>
	<span class="token label function">var3:</span> word <span class="token number">0</span>  	#var3 contiendra le résultat

.text
.globl main
<span class="token label function">main:</span>
	la <span class="token number">$11</span>, var1
	la <span class="token number">$12</span>, var2
	la <span class="token number">$13</span>, var3
	
	lw <span class="token number">$21</span>, <span class="token number">0</span>(<span class="token number">$11</span>)
	lw <span class="token number">$22</span>, <span class="token number">0</span>(<span class="token number">$12</span>)
	add <span class="token number">$23</span>, <span class="token number">$22</span>, <span class="token number">$21</span>
	sw <span class="token number">$23</span>, <span class="token number">0</span>(<span class="token number">$13</span>)
	
	li <span class="token number">$2</span>, <span class="token number">10</span>
	syscall
</code></pre>
<p><em><strong>Question:</strong></em> <em>A quoi correspond la première instruction du segment <code>text</code>?</em></p>
<p><em><strong>Question:</strong></em> <em>A quelles adresses sont rangées les trois variables <code>var1</code>, <code>var2</code> et <code>var3</code>?</em></p>
<p><code>var1</code> est rangé à l’adresse 0x1001000, <code>var2</code> en 0x1001004 et <code>var3</code> en 0x1001008.</p>
<p><em><strong>Question:</strong></em> <em>Lancer l’exécution du programme en mode pas à pas, en analysant pour chaque instruction quels registres du processeur et quelles cases de la mémoire externe ont été modifiées.</em></p>
<pre class=" language-nasm"><code class="prism  language-nasm">.data
	<span class="token label function">var1:</span> word <span class="token number">0x12</span>		# valeur de var1: <span class="token number">0x00000012</span> à l'adresse mémoire <span class="token number">0x1001000</span>
	<span class="token label function">var2:</span> word <span class="token number">0x34</span>		# valeur de var2: <span class="token number">0x00000034</span> à l'adresse mémoire <span class="token number">0x1001004</span>
	<span class="token label function">var3:</span> word <span class="token number">0</span>		# valeur de var3: <span class="token number">0x00000000</span> à l'adress mémoire <span class="token number">0x1001008</span>

.text
.globl main
<span class="token label function">main:</span>
	la <span class="token number">$11</span>, var1		# <span class="token number">$1</span> et <span class="token number">$11</span> reçoivent <span class="token number">0x1001000</span> (adresse de var1)
	la <span class="token number">$12</span>, var2		# <span class="token number">$12</span> reçoit <span class="token number">0x1001004</span> (adresse de var2)
	la <span class="token number">$13</span>, var3		# <span class="token number">$13</span> reçoit <span class="token number">0x1001008</span> (adresse de var3)
	
	lw <span class="token number">$21</span>, <span class="token number">0</span>(<span class="token number">$11</span>)		# <span class="token number">$21</span> reçoit <span class="token number">0x00000012</span> (valeur de var1, contenue à l'adresse stockée dans <span class="token number">$11</span>)
	lw <span class="token number">$22</span>, <span class="token number">0</span>(<span class="token number">$12</span>)		# <span class="token number">$22</span> reçoit <span class="token number">0x00000034</span> (valeur de var2, contenue à l'adresse stockée dans <span class="token number">$12</span>)
	add <span class="token number">$23</span>, <span class="token number">$22</span>, <span class="token number">$21</span>	# <span class="token number">$23</span> reçoit <span class="token number">0x00000046</span> (somme de var1 et var2, dont les valeurs sont stockees dans <span class="token number">$21</span> et <span class="token number">$22</span>)
	sw <span class="token number">$23</span>, <span class="token number">0</span>(<span class="token number">$13</span>)		# on stocke dans l'adresse contenue (donc dans var3) dans <span class="token number">$13</span> la valeur stockée dans <span class="token number">$23</span>
	
	li <span class="token number">$2</span>, <span class="token number">10</span>			#exit(<span class="token number">0</span>)
	syscall
</code></pre>
<h2 id="somme-des-dix-premiers-entiers">2. Somme des dix premiers entiers</h2>
<p><strong>Code de la fonction <code>sum10</code></strong></p>
<pre class=" language-nasm"><code class="prism  language-nasm">.text
	li <span class="token number">$5</span>,<span class="token number">0</span>		#résultat
	li <span class="token number">$2</span>,<span class="token number">0</span>		#compteur de boucle
	li <span class="token number">$6</span>,<span class="token number">11</span>	#borne supérieure de la boucle

j  and 
<span class="token label function">for:</span>
	addu <span class="token number">$5</span>,<span class="token number">$5</span>,<span class="token number">$2</span>
	addiu  <span class="token number">$2</span>,<span class="token number">$2</span>,<span class="token number">1</span>

<span class="token label function">and:</span> bne <span class="token number">$2</span>,<span class="token number">$6</span>,for

li <span class="token number">$2</span>,<span class="token number">10</span>
syscall  
</code></pre>
<p><em><strong>Question:</strong></em> <em>Vérifier le résultat obtenu.</em><br>
Après exécution du code, la valeur stockée dans <code>$5</code> est 0x00000037, ce qui donne 55 en décimal. Le résultat est alors bien celui que l’on attendait.</p>
<h2 id="conversion-binaire---hexadécimal-ascii">3. Conversion binaire - hexadécimal ASCII</h2>
<p><em><strong>Question:</strong></em> <em>Ecrire le programme assembleur correspondant au programme C donné.</em></p>
<pre class=" language-nasm"><code class="prism  language-nasm">.data
	<span class="token label function">table:</span> .ascii  <span class="token string">"0123456789ABCDEF"</span>		# table des caractères ASCII des chiffres hexadécimaux
	string : .asciiz <span class="token string">"0x________"</span>			# chaîne de caractères résultat
	<span class="token label function">n:</span>  .word <span class="token number">0x5432ABCD</span>					# nombre à convertir

.text
	la <span class="token number">$3</span>,string							# <span class="token number">$3</span> reçoit l'adresse mémoire de string
	addiu <span class="token number">$3</span>,<span class="token number">$3</span>,<span class="token number">2</span>							# PS: on se décale de <span class="token number">2</span> octets pour se placer à la première case de string à remplir 
	li <span class="token number">$5</span>,<span class="token number">32</span>								# <span class="token number">$5</span> reçoit i <span class="token operator">=</span> <span class="token number">32</span>
	la <span class="token number">$7</span>,n									# <span class="token number">$7</span> reçoit l'adresse mémoire de n
	lw <span class="token number">$7</span>,<span class="token number">0</span>(<span class="token number">$7</span>)								# lecture en memoire de n et scotckage dans <span class="token number">$7</span>
	la <span class="token number">$8</span>,table								# <span class="token number">$8</span> reçoit l'adresse mémoire de table
 
<span class="token label function">do:</span>											
	addiu <span class="token number">$5</span>,<span class="token number">$5</span>,<span class="token number">-4</span>							# i <span class="token operator">=</span> i <span class="token operator">-</span> <span class="token number">4</span>
 	srav   <span class="token number">$6</span>, <span class="token number">$7</span>,<span class="token number">$5</span> 						# n <span class="token operator">&gt;</span><span class="token operator">&gt;</span> i
 	andi <span class="token number">$6</span>,<span class="token number">$6</span>,<span class="token number">0x000f</span>
 
	addu <span class="token number">$9</span>,<span class="token number">$8</span>,<span class="token number">$6</span>							#table<span class="token operator">+</span>q
	lb <span class="token number">$9</span>,<span class="token number">0</span>(<span class="token number">$9</span>)								# lecture en mémoire de table<span class="token operator">[</span>q<span class="token operator">]</span>
	sb <span class="token number">$9</span>,<span class="token number">0</span>(<span class="token number">$3</span>)								# ecriture memoire de table<span class="token operator">[</span>q<span class="token operator">]</span> dans <span class="token number">$3</span>
	addiu <span class="token number">$3</span>,<span class="token number">$3</span>,<span class="token number">1</span>							# PS<span class="token operator">+</span><span class="token operator">+</span>
 bgtz  <span class="token number">$5</span>,do 
 
	#AFFICHAGE
	la <span class="token number">$4</span>,string 
	li <span class="token number">$2</span>,<span class="token number">4</span>
	syscall 

	#SORTIE DE PROGRAMME
	li <span class="token number">$2</span>,<span class="token number">10</span>
	syscall 
</code></pre>

