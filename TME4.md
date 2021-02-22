*__COMPTE-RENDU LU3IN004 n°4:__* *AGUINI Lydia (3874863), GIANG Cécile (3530406)*

# ALMO TP n°4 - Génération de code avec GCC et exécution sur architecture matérielle mono-processeur

## 3. Exécution du code

*__Question:__* *Dans le fichier `app.bin.txt`, trouvez à quelle adresse débute le code la fonction `main()` dans le segment `seg_code`. Regardez maintenant le contenu du premier mot mémoire du segment `seg_data`. Qu'observez vous ?*

Le code de la fonction `main()` débute à l'adresse mémoire `0x00400000`. Nous observons que le premier mot mémoire du segment `seg_data` contient l'adresse en mémoire du `main()`.

```nasm
Disassembly of section seg_data:

10000000 <HexaTab.1077-0x74>:
10000000:	00400000 	0x400000		/* adresse e, mémoire du main*/
10000004:	6c65480a 	0x6c65480a
10000008:	57206f6c 	bnezl	t9,1001bdbc <HexaTab.1077+0x1bd48>
```
*__Question:__* *Pour les fonctions caractérisées par l'attribut `constructor (__attribute__((constructor)))`, `GCC` créé automatiquement une section nommée `.ctors` contenant un tableau de pointeurs vers ces fonction. En analysant le fichier `app.ld`, expliquez l'observation que vous avez faite à la question précédente.*

**Code de `app.ld`**

```nasm
INCLUDE seg.ld

 * Grouping sections into segments
 */

SECTIONS
{
    . = seg_code_base;
    seg_code : {
        *(.text)
    }
    . = seg_data_base;
    seg_data : {
        *(.ctors)
        *(.rodata)
        /* . = ALIGN(4); */
        *(.rodata.*)
        /* . = ALIGN(4); */
        *(.data)
        /* . = ALIGN(4); */
        *(.lit8)
        *(.lit4)
        *(.sdata)
        /* . = ALIGN(4); */
        *(.bss)
        *(COMMON)
        *(.sbss)
        *(.scommon)
    }
}
```

`GCC` crée la section `.ctors` contenant un tableau de pointeurs vers les fonctions caractérisées par l'attribut `constructor (__attribute__((constructor)))`, il va donc chercher dans le segment `sg_code` de telles fonctions; la première qui apparaît dans cette section étant `main()`, `GCC` stocke son adresse en premier dans la section `.ctors`.

## 4. Automatisation de la génération: écriture d'un Makefile

**Code du `Makefile`**

```nasm
CC = mipsel-unknown-elf-gcc
AS = mipsel-unknown-elf-as
LD = mipsel-unknown-elf-ld
DU = mipsel-unknown-elf-objdump

SYS_OBJS = reset.o \
		   giet.o \
		   common.o \
		   ctx_handler.o \
		   drivers.o \
		   exc_handler.o \
		   irq_handler.o \
		   sys_handler.o


APP_OBJS = stdio.o main.o

GIET ?= ../../giet

SYS_PATH = $(GIET)/sys
APP_PATH = $(GIET)/app

SYS_CFLAGS = -Wall -ffreestanding -mno-gpopt -mips32 -I$(SYS_PATH) -I.
APP_CFLAGS = -Wall -ffreestanding -mno-gpopt -mips32 -I$(APP_PATH) -I.

all: sys.bin app.bin

## system compilation

# les fichiers à générer sont :
# * sys.bin
# * reset.o et giet.o

giet.o : $(SYS_PATH)/giet.s
	$(AS) -mips32 -o $@ $< 

reset.o : reset.s 
	$(AS) -mips32 -o $@ $< 
	
# * les autres fichiers .o de $(SYS_OBJS) issus de fichiers .c
#  * pour la génération des ces fichiers, utilisez la variable $(SYS_CLAGS).

%.o : $(SYS_PATH)/%.c
	$(CC) $(SYS_CFLAGS) -c -o $@ $< 


sys.bin: sys.ld $(SYS_OBJS)
	$(LD) -o $@ -T $?

## app compilation

# les fichiers à générer sont :
# * app.bin
# * stdio.o et main.o
#  * pour la génération des ces fichiers, utilisez la variable $(APP_CLAGS).
	
main.o : main.c
	$(CC) $(APP_CFLAGS) -c -o $@ $<

stdio.o : $(APP_PATH)/stdio.c
	$(CC) $(APP_CFLAGS) -c -o $@ $<
	
app.bin : app.ld $(APP_OBJS)
	$(LD) -o $@ -T $?




### special rules

clean:
	rm -f *.o *.bin proc0*
```
