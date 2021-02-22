*__COMPTE-RENDU LU3IN004 n°8:__* *AGUINI Lydia (3874863), GIANG Cécile (3530406)*

# ALMO TP n°8 - Périphériques orientés blocs

## 1. Application séquentielle sans DMA

*__Question:__* *Complétez le fichier `reset.s` de façon à installer dans le vecteur d'interruption les 4 ISR correspondant aux quatre périphériques présents dans la plateforme matérielle.*

**Code rajouté:**

```nasm
    la      $27,    _isr_ioc            /* $27 <= isr_ioc address */
    sw      $27,    0*4($26)            /* interrupt_vector[0] <= _isr_ioc */
    la      $27,    _isr_dma           /* $27 <= isr_dma address */
    sw      $27,    2*4($26)            /* interrupt_vector[2] <= _isr_dma */
```

*__Question:__* *Complétez le fichier `main.c` qui contient la fonction principale `main()`.*

**Code complété du `main.c`**:

```c
#define NBLOCS 32
#define THRESHOLD 200

__attribute__((constructor)) void main(void)
{
    char buf_in[128*128];
    char buf_out[128*128];
    unsigned int i;
    unsigned int base = 0;

    while (base < 5 * NBLOCS)
    {

        tty_printf("\n *** image %d *** at date = %d \n",
                base / NBLOCS, proctime());

        /* Phase 1 : lecture image sur le disque et transfert vers buf_in */
        if (ioc_read(base,&buf_in,NBLOCS))
        {
            tty_printf("echec io_read\n");
            exit();
        }
        else
        {
            ioc_completed();
            tty_printf("io_read  completed at date = %d \n", proctime());
        }

        /* Phase 2 : transfert de buf_in vers buf_out avec seuillage */
        for (i = 0; i < 128 * 128; i++)
        {
            if (buf_in[i] > THRESHOLD)
                buf_out[i] = 255;
            else
                buf_out[i] = buf_in[i];
        }
        tty_printf("image processing completed at date = %d \n", proctime());

        /* Phase 3 : transfert de buf_out vers le frame buffer par dma */
        if (fb_sync_write(0, buf_out, 128*128))
        {
            tty_printf("echec fb_write\n");
            exit();
        }
        else
        {
            tty_printf("transfer completed at date = %d \n", proctime());
        }

        base = base + NBLOCS;
    }

    exit();
}
```

*__Question:__* *En quoi consiste la modification de l'image réalisée par cette application avant affichage ?*

La modification de l'image avant l'affichage consiste à rendre tous les pixels de niveau de gris supérieurs à 200 blancs (niveau de gris à 255).

*__Question:__* *Relevez les dates correspondant aux événements significatifs.*

**`image 0:`** at date = 455  cycles = 0 Kcycles
- `io_read` complet à date = 156020 cycles = 156 Kcycles
- traitement d'image complet à date = 463993 cycles = 464 Kcycles
- transfert complet à date 584182 cycles = 584 Kcycles 

**`image 1:`** at date = 587439  cycles = 587 Kcycles
- `io_read` complet à date = 771369 cycles = 771 Kcycles
- traitement d'image complet à date = 1079306 cycles = 1079 Kcycles
- transfert complet à date 1199427 cycles = 1199 Kcycles 

**`image 2:`** at date = 1202770  cycles = 1203 Kcycles
- `io_read` complet à date = 1399161 cycles = 1399 Kcycles
- traitement d'image complet à date = 1707184 cycles = 1707 Kcycles
- transfert complet à date 1827305 cycles = 1827 Kcycles 

**`image 3:`** at date = 1830648  cycles = 1831 Kcycles
- `io_read` complet à date = 1876704 cycles = 1877 Kcycles
- traitement d'image complet à date = 2184727 cycles = 2185 Kcycles
- transfert complet à date 2304848 cycles = 2305 Kcycles 

**`image 4:`** at date = 2308191  cycles = 2308 Kcycles
- `io_read` complet à date = 2435704 cycles = 2436 Kcycles
- traitement d'image complet à date = 2743625 cycles = 2744 Kcycles
- transfert complet à date 2863746 cycles = 2864 Kcycles 


*__Question:__* *Construisez un tableau contenant, pour chacune des 5 images, la durée de la phase de chargement, la durée de la phase de traitement, et la durée de la phase d'affichage.*


|         |  Chargement | Traitement  | Affichage   |
|---------|-------------|-------------|-------------|
| image 0 | 156 Kcycles | 308 Kcycles | 120 Kcycles |
| image 1 | 184 Kcycles | 308 Kcycles | 120 Kcycles |
| image 2 | 197 Kcycles | 308 Kcycles | 120 Kcycles |
| image 3 | 46 Kcycles  | 308 Kcycles | 120 Kcycles |
| image 4 | 127 Kcycles | 308 Kcycles | 120 Kcycles |


## 2. Application séquentielle sans DMA


**_Question:_** *Sauvez le fichier main.c sous un autre nom, puis éditez-le et modifiez l'application pour que la phase `_Display_` utilise dorénavant l'appel système non-bloquant `fb_write()`, qui configure et démarre le contrôleur `DMA` pour que celui-ci effectue le transfert du tampon `buf_out` vers le tampon vidéo par rafales de 32 mots (128 octets). Notez que l'application logicielle doit maintenant utiliser l'appel système bloquant `fb_completed()` pour s'assurer que le transfert est terminé avant de passer à l'image suivante.*

**Comde complété du `main.c`**

```c
#define NBLOCS 32
#define THRESHOLD 200

__attribute__((constructor)) void main(void)
{
    char buf_in[128*128];
    char buf_out[128*128];
    unsigned int i;
    unsigned int base = 0;

    while (base < 5 * NBLOCS)
    {

        tty_printf("\n *** image %d *** at date = %d \n",
                base / NBLOCS, proctime());

        /* Phase 1 : lecture image sur le disque et transfert vers buf_in */
        if (ioc_read(base,&buf_in,NBLOCS))
        {
            tty_printf("echec io_read\n");
            exit();
        }
        else
        {
            ioc_completed();
            tty_printf("io_read  completed at date = %d \n", proctime());
        }

        /* Phase 2 : transfert de buf_in vers buf_out avec seuillage */
        for (i = 0; i < 128 * 128; i++)
        {
            if (buf_in[i] > THRESHOLD)
                buf_out[i] = 255;
            else
                buf_out[i] = buf_in[i];
        }
        tty_printf("image processing completed at date = %d \n", proctime());

        /* Phase 3 : transfert de buf_out vers le frame buffer par dma */
        if (fb_write(0, &buf_out, 128*128))
        {
            tty_printf("echec fb_write\n");
            exit();
        }
        else
        {
            tty_printf("transfer completed at date = %d \n", proctime());
        }
        fb_completed();
	
        base = base + NBLOCS;
	
    }

    exit();
}
```
**Question:** *Relevez les dates correspondant aux événements significatifs, et reconstruisez un tableau similaire à celui de la partie précedente.*

**`image 0:`** at date = 0 Kcycles
- `io_read` complet à date = 156 Kcycles
- traitement d'image complet à date = 464 Kcycles
- transfert complet à date = 481 Kcycles 

**`image 1:`** at date = 484 Kcycles
- `io_read` complet à date = 668 Kcycles
- traitement d'image complet à date = 976 Kcycles
- transfert complet à date = 993 Kcycles 

**`image 2:`** at date = 996 Kcycles
- `io_read` complet à date = 1192 Kcycles
- traitement d'image complet à date 1500 Kcycles
- transfert complet à date 1517 Kcycles 

**`image 3:`** at date = 1520 Kcycles
- `io_read` complet à date = 1566 Kcycles
- traitement d'image complet à date = 1874 Kcycles
- transfert complet à date 1891 Kcycles 

**`image 4:`** at date = 1894 Kcycles
- `io_read` complet à date = 2022 Kcycles
- traitement d'image complet à date = 2330 Kcycles
- transfert complet à date = 2347 Kcycles

|         | Chargement  | Traitement  | Affichage   |
|---------|-------------|-------------|-------------|
| image 0 | 156 Kcycles | 308 Kcycles | 17 Kcycles |
| image 1 | 184 Kcycles | 308 Kcycles | 17 Kcycles |
| image 2 | 196 Kcycles | 308 Kcycles | 17 Kcycles |
| image 3 | 46 Kcycles  | 308 Kcycles | 17 Kcycles |
| image 4 | 128 Kcycles | 308 Kcycles | 17 Kcycles |

Cette fois-ci nous remarquons que les transferts prennent beaucoup moins de cycle, ce qui fait considérablement baisser la durée d'affichage (on passe de 120 à 17 cycles !).

## 3. Application pipe-line

**Question:** *Relevez les dates permettant de déterminer les durées d'exécution.*


**`image 0:`** 
- `io_read` complet à date = 152 Kcycles
- traitement d'image complet à date = 644 Kcycles
- transfert complet à date = 1231 Kcycles 

**`image 1:`**
- `io_read` complet à date = 336 Kcycles
- traitement d'image complet à date = 1214 Kcycles
- transfert complet à date = 1602 Kcycles 

**`image 2:`** 
- `io_read` complet à date = 840 Kcycles
- traitement d'image complet à date = 1585 Kcycles
- transfert complet à date 2120 Kcycles 

**`image 3:`** 
- `io_read` complet à date = 1277 Kcycles
- traitement d'image complet à date = 2103 Kcycles
- transfert complet à date 2445 Kcycles 

**`image 4:`** 
- `io_read` complet à date = 1729 Kcycles
- traitement d'image complet à date = 2428 Kcycles
- transfert complet à date = 2461 Kcycles

**_Question:_** *Quel est le temps de traitement d'une image quand on est en régime établi ?*

Le traitement d'une image dure entre 307 et 374 Kcycles.
