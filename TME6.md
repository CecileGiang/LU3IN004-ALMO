*__COMPTE-RENDU LU3IN004 n°6:__* *AGUINI Lydia (3874863), GIANG Cécile (3530406)*

# ALMO TP n°6 - Mémoires Cache: mesures de performance

## 1. Caches de faible capacité

*__Question:__* *Commencez par compiler le système logiciel à l'aide du `Makefile` fourni, puis lancez le simulateur avec des caches L1 (instructions et données) dont la capacité est d'une seule ligne de cache (une seule case), en activant la génération des statistiques. Chaque cache a donc une capacité de 16 octets. Quel est le CPI (nombre moyen de _Cycles Par Instruction_) pour ces tailles de cache ?*

Le CPI pour `NICACHES = 1` et `NDCACHE = 1` est de 5.999188.

*__Question:__* *Quel est le temps d'exécution de l'application (en nombre de cycles) ?*

Le temps d'exécution de l'application est de 3593454 cycles.

*__Question:__* *Relancez la simulation en doublant la capacité des caches à 32 octets (2 cases). Que constatez vous concernant le temps d'exécution et le CPI ?*

On a maintenant CPI =  5.204033, l'application s'exécute en 3117191 cycles, ce qui est plus rapide que pour une capacité d'une seule ligne de cache (une seule case), ce qui est normal: le cache pourra stocker plus d'informations et donc le processeur y aura accès plus rapidement que s'il était obligé d'accéder en permanence à la mémoire.

## 2. Caches de capacité "normale"

*__Question:__* *Faites varier la taille du cache d'instructions de 4 à 1024 cases par puissance de 2, et notez à chaque fois le CPI et le temps de calcul obtenus. Vous utiliserez comme taille de cache de données NDCACHE = 1024 (cache de 16 kibi octets).*

**En faisant varier la taille du cache d’instructions avec `NDCACHE = 1024`:**

__*4 cases:*__ 
- 2161003 cycles
- CPI = 3.608316


__*8 cases:*__
- 1791705 cycles
- CPI = 2.991187

__*16 cases:*__
- 1595413 cycles
- CPI = 2.663487

__*32 cases:*__
- 1494123 cycles
- CPI = 2.494386

__*64 cases:*__
- 1153000 cycles 
- CPI = 1.924888
 
__*128 cases:*__
- 915979 cycles
- CPI = 1.529190

__*256 cases:*__
- 841396 cycles
- CPI = 1.421376
 
 __*512 cases:*__
 - 831596 cycles
 - CPI = 1.388320
 
 __*1024 cases:*__
- 831596 cycles
- CPI = 1.388320

**Variation de la taille du cache de données avec `NICACHE = 1024`:**

__*1 case:*__
- 2217436 cycles
- CPI= 3.701948

__*2 cases:*__
- 1702888 cycles
- CPI= 2.842928

__*4 cases:*__
- 1317363 cycles
 CPI = 2.199291

__*8 cases:*__
- 1167179 cycles
- CPI = 1.948573

__*16 cases:*__
- 1027337 cycles
- CPI = 1.715107

__*32 cases:*__
- 916328 cycles
- CPI = 1.529780

__*64 cases:*__
- 842454 cycles 
- CPI = 1.406446
 
__*128 cases:*__
- 835001 cycles
- CPI = 1.394002

__*256 cases:*__
- 831596 cycles
- CPI = 1.388320
 
 __*512 cases:*__
 - 831596 cycles
 - CPI = 1.388320
 
 __*1024 cases:*__
- 831596 cycles
- CPI = 1.388320

_Note personnelle:_ 
```nasm
commande: grep "0fjndksl" stats
```


## 3. Caches désactivés

*__Question:__* *Relancez l'exécution du simulateur en désactivant les deux caches, c'est-à-dire en utilisant les valeurs : `NICACHE = NDCACHE = 0`. Quelle est la durée d'exécution du programme ? Quelle est la valeur du CPI ?*

Pour `NICACHE = NDCACHE = 0`
- 5294329 cycles
- CPI = 8.789221


## 4. Représentation graphique

[Graphe du CPI en fonction de NDCACHE](https://ibb.co/JFtn6vP)

[Graphe du CPI en fonction de NICACHE](https://ibb.co/Q6PpN2K)
