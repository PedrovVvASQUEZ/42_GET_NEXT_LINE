# get_next_line

Projet C de l’école 42 : lire un flux progressivement et renvoyer son contenu ligne par ligne, sans charger le fichier entier en mémoire.

## Fonctionnement

La fonction `get_next_line` lit un descripteur avec `read(2)` et conserve les caractères déjà lus mais pas encore retournés. Chaque appel renvoie la prochaine ligne, jusqu’au prochain `\n` inclus. À la fin du flux, elle renvoie le dernier fragment s’il ne se termine pas par un saut de ligne, puis renvoie `NULL` aux appels suivants. Elle renvoie également `NULL` en cas de descripteur invalide ou d’erreur de lecture.

La mémoire de chaque ligne retournée appartient à l’appelant et doit être libérée avec `free`.

## Versions

- **Partie obligatoire** : un état de lecture conservé pour un descripteur à la fois.
- **Partie bonus** : un état distinct par descripteur, permettant d’alterner les appels sur plusieurs fichiers ouverts. L’état est stocké dans un tableau statique de 1024 entrées, indexé par la valeur du descripteur.

Les deux versions exposent la même fonction. Il faut donc compiler **soit** les fichiers obligatoires, **soit** les fichiers bonus, et ne pas les lier ensemble.

## Configuration

`BUFFER_SIZE` détermine le nombre d’octets lus à chaque appel de `read`. Sa valeur par défaut dans les en-têtes est `6` ; elle peut être remplacée à la compilation, par exemple avec `-D BUFFER_SIZE=42`.

## Utilisation

Exemple de programme client, placé à la racine du dépôt dans `main.c` :

```c
#include "get_next_line/get_next_line.h"
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(void)
{
    int     fd;
    char    *line;

    fd = open("file.txt", O_RDONLY);
    if (fd < 0)
        return (1);
    while ((line = get_next_line(fd)) != NULL)
    {
        printf("%s", line);
        free(line);
    }
    close(fd);
    return (0);
}
```

Compiler la partie obligatoire depuis la racine du dépôt :

```sh
cc -Wall -Wextra -Werror -D BUFFER_SIZE=42 main.c \
  get_next_line/get_next_line.c \
  get_next_line/get_next_line_utils.c \
  -o get_next_line_app
```

Pour compiler la partie bonus, remplacer les deux fichiers `.c` obligatoires par `get_next_line/get_next_line_bonus.c` et `get_next_line/get_next_line_utils_bonus.c`.

## Organisation du dépôt

```text
get_next_line/
├── get_next_line.c                 # Implémentation obligatoire
├── get_next_line.h                 # En-tête obligatoire et BUFFER_SIZE
├── get_next_line_utils.c           # Fonctions utilitaires obligatoires
├── get_next_line_bonus.c           # Implémentation multi-descripteurs
├── get_next_line_bonus.h           # En-tête bonus
└── get_next_line_utils_bonus.c     # Fonctions utilitaires bonus
```

Les fonctions utilitaires fournissent notamment les opérations de longueur, recherche, duplication, concaténation et extraction de sous-chaînes nécessaires à la lecture et à la découpe des lignes.

## État du dépôt

Le dépôt contient les sources des deux versions, mais pas de `Makefile`, de programme principal ni de tests automatisés. Les commandes de compilation ci-dessus supposent donc qu’un fichier client `main.c` est fourni.

### Points d’attention

- La version bonus indexe son tableau d’état directement avec `fd`. Elle vérifie les descripteurs négatifs, mais ne vérifie pas que `fd` est inférieur à 1024 ; utiliser un descripteur de valeur supérieure peut donc accéder hors du tableau.
- Les fichiers `.c` bonus incluent actuellement `get_next_line.h` plutôt que `get_next_line_bonus.h`. Les déclarations des deux en-têtes sont similaires, mais l’en-tête bonus n’est donc pas celui utilisé directement par ces fichiers sources.
