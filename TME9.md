*__COMPTE-RENDU LU3IN004 n°9:__* *AGUINI Lydia (3874863), GIANG Cécile (3530406)*

# ALMO TP n°9 - Fonctionnement Multi-Tâches

*__Question 1:__* *Ouvrez les fichiers `main0.c`, `main1.c`, `main2.c` et `main3.c` pour déterminer ce que fait chacune de ces quatre applications qui vont s'exécuter en parallèle.*

- **`main0.c`:** Cette fonction compte le nombre de cycles écoulés par l'appel des fonctions `tty_getc_irq` qui lit un caractère au clavier et le stocke dans la variable `buf` *(elle contient une boucle de scrutation qui s'exécute en mode utilisateur, et dans laquelle on appelle de façon répétée la fonction système `_tty_read_irq` par l'intermédiaire d'un appel système)* et `tty_printf` qui affiche le temps CPU écoulé en secondes dans le processus courant. Enfin elle boucle.

- **`main1.c`:**  Cette fonction calcule et affiche la factorielle d'un entier non signé `opx` entré sur clavier sous la condition qu'il soit compris dans `[1, 12]`. Sinon, elle affiche un message d'erreur. Enfin, elle boucle.

- **`main2.c`:** Cette fonction calcule et affiche le pgcd de deux entiers non signés `opx` et `opy` entrés au clavier. Enfin elle boucle.

- **`main3.c`:** Cette fonction lit une image depuis le disque, la traite avec une opération de seuillage, puis l'envoie sur le frame buffer pour qu'il affiche à l'écran. Enfin, elle boucle pour refaire ça avec 19 autres images, puis elle s'interrompt.


*__Question 2:__* *Compléter le fichier `reset.s` pour qu'il réalise les opérations suivantes:*
*1.  Installation des sept ISR dans le vecteur d'interruption*
*2.  Initialisation des registres pour la tâche  T0*
*3.  Initialisation des tableaux de contexte pour les tâches  T1,  T2  et  T3*
*4.  Configuration des composants TIMER et ICU.*

**Code de `reset.s`**

```nasm
/*
    .section .reset, "ax", @progbits

    .func   reset
    .type   reset, %function

reset:
    .set noreorder

    /* initializes interrupt vector */
    la      $27,    _interrupt_vector   /* $27 <= interrupt vector address */
    la      $26,    _isr_switch
    sw      $26,    1*4($27)            /* interrupt_vector[1] <= _isr_switch */
    la      $26,    _isr_ioc
    sw      $26,    0*4($27)              /* interrupt_vector[0] <= _isr_ioc */
    la      $26,    _isr_dma
    sw      $26,    2*4($27)  /* interrupt_vector[2] <= _isr_dma */
    la      $26,    _isr_tty_get_task0
    sw      $26,    3*4($27)  /* interrupt_vector[3] <= _isr_tty_get_task0 */
    la      $26,    _isr_tty_get_task1
    sw      $26,    4*4($27)  /* interrupt_vector[4] <= _isr_tty_get_task1 */
    la      $26,    _isr_tty_get_task2
    sw      $26,    5*4($27)   /* interrupt_vector[5] <= _isr_tty_get_task2 */
    la      $26,    _isr_tty_get_task3
    sw      $26,    6*4($27)   /* interrupt_vector[6] <= _isr_tty_get_task3 */

    /* initializes task index and task number */
    la      $27,    _current_task_array
    sb      $0,     0($27)              /* task_index <= 0 (not necessary, already done by default) */
    la      $27,    _task_number_array
    li      $26,    4
    sb      $26,    0($27)              /* task_number <= 4 */

    /* initializes stack pointers for tasks 1,2,3 (each stack is 64 KB) */
    la      $27,    _task_context_array /* $27 <= &ctx[] */
    la      $26,    seg_stack_base
    li      $10,    0x00020000          /* 128 K */
    add     $10,    $26,    $10
    sw      $10,    4*(64+29)($27)      /* SP for task 1 */
    li      $10,    0x00030000          /* 192 K */
    add     $10,    $26,    $10
    sw      $10,    4*(128+29)($27)     /* SP for task 2 */
    li      $10,    0x00040000     /* 256 K */
    add     $10,    $26,    $10
    sw      $10,    4*(192+29)($27)     /* SP for task 3 */

    /* initializes EPC for tasks 1,2,3 (main1, main2, main3) */
    la      $26,    seg_data_base
    lw      $10,    1*4($26)            /* retrieves main1 address */
    sw      $10,    4*(64+32)($27)      /* EPC for task 1 */
    lw      $10,    2*4($26)            /* retrieves main2 address */
    sw      $10,    4*(128+32)($27)     /* EPC for task 2 */
    lw      $10,    3*4($26)     /* retrieves main3 address */
    sw      $10,    4*(192+32)($27)      /* EPC for task 3 */

    /* initializes RA for tasks 1,2,3 (to execute the eret instruction) */
    la      $26,    to_user
    sw      $26,    4*(64+31)($27)      /* $31 for task 1 */
    sw      $26,    4*(128+31)($27)     /* $31 for task 2 */
    sw      $26,    4*(192+31)($27)      /* $31 for task 3 */

    /* initializes SR sor tasks 1,2,3 (set IM, UM, EXL, IE) */
    li      $26,    0x0000FF13
    sw      $26,    4*64($27)           /* SR for task 1 */
    sw      $26,    4*128($27)          /* SR for task 2 */
    sw      $26,    4*192($27)      /* SR for task 3 */

    /* ICU Configuration: 7 IRQs enabled (IOC, TIMER, DMA, TTYs) */
    la      $27,    seg_icu_base
    li      $26,    0x0000007F
    sw      $26,    2*4($27)            /* ICU_MASK_SET = 0111 1111 */

    /* TIMER Configuration: period = 10000 cycles */
    la      $27,    seg_timer_base
    li      $26,    10000               /* CHANGER CETTE VALEUR POUR FAIRE VARIER LE TIMER */
    sw      $26,    8($27)            /* period <= 10000 */
    li      $26,    0x3
    sw      $26,    1*4($27)            /* TIMER start */

    /* initializes registers for T0 (stack size = 64K) */
    la      $27,    seg_stack_base
    li      $26,    0x0010000
    add     $29,    $27,    $26         /* SP for task 0 */
    la      $26,    seg_data_base
    lw      $26,    0($26)              /* retrieves main0 address */
    mtc0    $26,    $14                 /* EPC for task 0 */
    li      $26,    0x0000FF13          /* set IM, UM, EXL and IE */
    mtc0    $26,    $12                 /* SR for task 0 */

    /* jumps to address contained in EPC (in user mode) */
to_user:
    eret

    .set reorder

    .endfunc
    .size reset, .-reset
```

*__Question 3:__* *Comment est-on sûr qu'il n'y aura pas d'interruptions pendant toute la phase d'initialisation du système (c'est-à-dire pendant qu'on exécute le code de démarrage) ? Quand et comment le processeur redevient-il interruptible ?*

On est sûrs qu'il n'y aura pas d'interruption pendant la phase d'initialisation du système car le masque de l'ICU n'est pas initialisé???????

*__Question 4:__* *Compilez l'application logicielle multi-tâches en exécutant le `Makefile` fourni, puis lancez la simulation. Si vous n'avez pas commis d'erreurs dans le code du `reset`, vous devez constater que les quatre applications s'exécutent en parallèle.*

Tout marche bien.

*__Question 5:__* *En analysant le contenu du fichier `sys.bin.txt`, évaluer le nombre d'instructions exécutées lors d'un changement de contexte (entre le branchement au **GIET** déclenchée par l'interruption, et la reprise de l'exécution de la tâche entrante).*

- Liste des fonctions appelées, ainsi que les instructions effectuées dans elles:
**_giet :** 8  instructions
**_int_handler :** 25 instructions 
**_int_demux :** 9  instructions
**_icu_read : ** 32 instructions** **_procid :** 11 instructions
**_int_demux :** 13 instructions
