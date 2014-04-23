---
layout: post
title: "Les constantes en C"
date: 2014-04-23 02:01:53 +0200
author: Baptiste Fontaine
comments: true
categories:
- C
- programmation
---

Il y a différentes façon d’avoir de définir des constantes (entières) en C,
mais la meilleure d’entre elles est probablement la moins connue.

<!--more-->

## `#define`

La méthode la plus simple consiste à utiliser `#define` pour avoir un
remplacement effectué lors du pré-processing.

Par exemple si l’on considère le bout de programme suivant :

```c
#define MAGIC 42
int main(void) {
        printf("%d\n", MAGIC);
        return 0;
}
```

Après pré-processing (`gcc -E`), le code devient :

```c
int main(void) {
        printf("%d\n", 42);
        return 0;
}
```

Si cette méthode est rapide et facile, elle comporte néanmoins un désavantage :
le nom de la constante n’est plus visible après le pré-processing, donc les
messages du compilateur ne comprendront que sa valeur, pas son nom (donc ici,
`42` au lieu de `MAGIC`).

## `const`

Une autre méthode consiste à utiliser le mot-clef `const`, qui est justement
fait pour déclarer des constantes.

```c
const int MAGIC = 42;
```

Cette méthode règle le problème du nom mais ici `MAGIC` n’est pas traitée comme
une vrai constante au moment de la compilation, et donc ne peut pas être
utilisé dans un `switch` et complique les optimisations.

Le bout de programme suivant ne compilera pas sans <i>warning</i> :

```c
switch(42) {
case MAGIC:
        puts("ok");
}
```

```
$ gcc -pedantic test.c
test.c:10:14: warning: expression is not an integer constant expression; folding it to a constant is a GNU extension [-Wgnu-folding-constant]
        case MAGIC:
             ^~~~~
1 warning generated.
```

À noter que problème est spécifique à C, il ne survient pas en C++.

## Un `enum` à un seul membre

La dernière solution pour notre problème est d’utiliser un `enum` avec un seul
membre.

```c
enum {
    MAGIC = 42
};
```

Cela peut paraître [étrange][ptest] au premier abord, mais cette technique
règle les problèmes des deux méthodes précédentes : on obtient une vrai
constante au moment de la compilation (ce qui veut dire qu’on peut l’utiliser
partout où une valeur immédiate est attendue), et on conserve le symbole, ce
qui pourra être utile pour les messages du compilateur et le débogage avec GDB.

[ptest]: https://github.com/orangeduck/ptest/blob/master/ptest.c#L11-21

---

Note : cet article est issu d’un email de [Daniel Holden][holden] qui répondait
à ma question concernant son utilisation des `enum`s à un seul membre dans l’un
de ses projets.

[holden]: http://www.theorangeduck.com/page/about
