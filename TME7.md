*__COMPTE-RENDU LU3IN004 n°7:__* *AGUINI Lydia (3874863), GIANG Cécile (3530406)*

# ALMO TP n°7 - Communications par Interruptions

## 1. Concentrateur d'interruptions (ICU)

*__Question:__* *En analysant les trois instructions du code de démarrage qui réalisent la configuration du composant `ICU`, déterminez à quelle adresse est implanté le registre de configuration `ICU_MASK` de l'`ICU`*

Le segment des registres du périphérique ICU, à l'adresse `0x9F000000`, et on a :
```nasm
sw $27, 2*4($26)       /* ICU_MASK_SET = 0xA */
``` 
Donc  `ICU_MASK` est à l'adresse `0x9F000008`.


## 2. Contrôleur d'horloge programmable (TIMER) 

*__Question:__* *Déterminez quelles actions sont exécutées par l'`ISR` associée au `TIMER`*

L'`ISR` crée un tableau contenant les adresses des 8 timers utilisés (`timer_adress`), et 
met la case d'indice `TIMER_RESETIRQ` à 0 (reset IRQ à l'écriture de n'importe quelle valeur dans ce registre on acquitte l'interruption), affiche un message à chaque interruption du timer.

*__Question:__* *Complétez cette fonction pour définir une période de 5000000 cycles, puis activer le composant d'horloge.*

```nasm
timer_set_period(5000000);
timer_set_mode(0b11);
```
On observe une exception `Exception : illegal read address` et l'affichage de `EPC` et `BAR` (l'adresse erronée). On explique le fait que le `TTY` continue à afficher les messages par le fait que le processeur s'est branché dans une boucle infinie. Il ne peut sortir de cette boucle que si on redémarre.

## 3. Lecture de caractères sur le terminal TTY

*__Question:__* *Regardez le fichier  `irq_handler.c` et expliquez ce que fait le code de l'`ISR` `_isr_tty_get_task0()` (et par extension, le code de `_isr_tty_get_indexed()`) ? Que se passe-t-il si le tampon de réception n'est pas vide lorsque l'`ISR` est exécutée ?*

La fonction `_isr_tty_get_task0()` reconnait les `IRQ` en utilisant les adresses de bases des terminaux et les task-id (0,1,2,3), et elle établit donc une communication pour terminal et manipule le tampon mémoire par la variable `_tty_get_full` (set/reset) et calcule l'indice du terminal avec `tty_id = proc_id * ntasks + task_id`. La fonction `_isr_tty_get_indexed()` sauvegarde le caractère, réinitialise l'`IRQ` et signale un caractère valide.

Quand l'ISR est exécutée si le tampon n'est pas vide le caractère est perdu.

*__Question:__* *Ouvrez le fichier `stdio.c`, et expliquez ce que fait l'appel système `tty_getc_irq()`*

L'appel système `tty_getc_irq()` extrait un caractère du terminal défini par le processeur identifiant et par task id on multi-tache. Il utilise donc `IRQ-GET interrupt` et le buffer associée au kernel, et retourne 0 en cas de succès.

*__Question:__* *Ouvrez le fichier `stdio.c`, et expliquez ce que fait l'appel système `_tty_read_irq()`*

Cet appel utilise `TTY-GET-IRQ` et le kernel buffer écrit par l'`ISR` , extrait un caractère du `TTY-get-buf[tty-index]`, l'écrit au `user buffer` et met donc la variable `tty-get-full[tty-index]` à 0, et retourne 0 si le `kernel buffer` est vide, 1 sinon.


**Edition du main:**

```nasm
# include <stdio.h>

__attribute__((constructor)) void main(void)
{
    unsigned int i;
    char n='1';

    timer_set_period(5000000);
    timer_set_mode(0b11);

    for (i = 0; i < 1000; i++)
    {
        if (tty_printf(" hello world : %d\n",i))
        {
            tty_puts("echec tty_printf\n");
            exit();
        }
	tty_getc_irq(&n);   
    }
    exit();
}
```
