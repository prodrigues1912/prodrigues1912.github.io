---
date: 2025-12-15
categories:
  - Programação
  - C
---

# Ponteiros em C

Explicação detalhada de ponteiros, um dos conceitos mais importantes da linguagem C.

<!-- more -->

!!! note "Pré-requisito"
    Este post assume conhecimento básico de C. Veja a [referência rápida de C](c-cheatsheet.md) para revisar o básico.

## O que é um ponteiro?

Um ponteiro é uma variável que armazena um **endereço de memória**. Em vez de guardar um valor diretamente (como `int x = 10`), o ponteiro guarda a localização onde esse valor está armazenado.

```c
int x = 10;      // x guarda o valor 10
int *ptr = &x;   // ptr guarda o endereço onde x está na memória
```

## Por que usar ponteiros?

1. **Passar dados por referência** - modificar variáveis dentro de funções
2. **Alocação dinâmica** - criar estruturas de tamanho variável em tempo de execução
3. **Estruturas de dados** - listas ligadas, árvores, grafos
4. **Performance** - evitar cópia de grandes estruturas
5. **Arrays e strings** - em C, são essencialmente ponteiros

## Declaração e operadores

```c
int x = 42;
int *ptr;     // declara ponteiro para int (ainda não aponta para nada)

ptr = &x;     // & = "endereço de" - ptr agora aponta para x

printf("%d\n", *ptr);  // * = "valor apontado por" - imprime 42
```

Dois operadores fundamentais:

| Operador | Nome | Significado |
|----------|------|-------------|
| `&` | Endereço de | Retorna o endereço de memória de uma variável |
| `*` | Desreferência | Acessa o valor no endereço apontado |

## Visualizando na memória

```
Memória:
┌─────────┬─────────┬─────────┬─────────┐
│  ...    │   42    │  ...    │  0x100  │
└─────────┴─────────┴─────────┴─────────┘
            ↑ x                  ↑ ptr
          0x100                0x200

x está no endereço 0x100 e contém 42
ptr está no endereço 0x200 e contém 0x100 (o endereço de x)
```

```c
#include <stdio.h>

int main(void) {
    int x = 42;
    int *ptr = &x;

    printf("Valor de x: %d\n", x);           // 42
    printf("Endereço de x: %p\n", &x);       // 0x7fff... (varia)
    printf("Valor de ptr: %p\n", ptr);       // mesmo endereço
    printf("Valor apontado: %d\n", *ptr);    // 42

    return 0;
}
```

## Modificando valores através de ponteiros

```c
int x = 10;
int *ptr = &x;

*ptr = 20;  // modifica x através do ponteiro

printf("%d\n", x);  // 20
```

Isso é útil para modificar variáveis dentro de funções:

```c
// Sem ponteiro - não funciona
void incrementa(int n) {
    n = n + 1;  // modifica apenas a cópia local
}

// Com ponteiro - funciona
void incrementa_ptr(int *n) {
    *n = *n + 1;  // modifica o valor original
}

int main(void) {
    int x = 10;

    incrementa(x);
    printf("%d\n", x);  // ainda 10

    incrementa_ptr(&x);
    printf("%d\n", x);  // agora 11

    return 0;
}
```

## Ponteiros e arrays

Em C, o nome de um array é um ponteiro para seu primeiro elemento:

```c
int arr[] = {10, 20, 30, 40, 50};
int *ptr = arr;  // equivalente a: int *ptr = &arr[0];

printf("%d\n", *ptr);        // 10 (primeiro elemento)
printf("%d\n", *(ptr + 1));  // 20 (segundo elemento)
printf("%d\n", *(ptr + 2));  // 30 (terceiro elemento)

// Notação de array também funciona com ponteiros
printf("%d\n", ptr[0]);  // 10
printf("%d\n", ptr[1]);  // 20
printf("%d\n", ptr[2]);  // 30
```

Visualização:

```
arr:  [10] [20] [30] [40] [50]
        ↑
       ptr
      ptr+0  ptr+1  ptr+2  ptr+3  ptr+4
```

## Aritmética de ponteiros

Quando você soma 1 a um ponteiro, ele avança para o próximo elemento do tipo, não para o próximo byte:

```c
int arr[] = {10, 20, 30};
int *ptr = arr;

printf("%p\n", ptr);      // 0x100 (exemplo)
printf("%p\n", ptr + 1);  // 0x104 (avança 4 bytes, tamanho de int)
printf("%p\n", ptr + 2);  // 0x108

// Diferença entre ponteiros
int *inicio = &arr[0];
int *fim = &arr[2];
printf("%ld\n", fim - inicio);  // 2 (elementos, não bytes)
```

## Ponteiros para ponteiros

Um ponteiro pode apontar para outro ponteiro:

```c
int x = 10;
int *ptr = &x;      // ponteiro para int
int **pptr = &ptr;  // ponteiro para ponteiro para int

printf("%d\n", x);       // 10
printf("%d\n", *ptr);    // 10
printf("%d\n", **pptr);  // 10

**pptr = 20;  // modifica x
printf("%d\n", x);  // 20
```

Visualização:

```
┌─────┐     ┌─────┐     ┌─────┐
│ 10  │ ←── │0x100│ ←── │0x200│
└─────┘     └─────┘     └─────┘
   x          ptr         pptr
 0x100       0x200       0x300
```

Uso comum: modificar um ponteiro dentro de uma função:

```c
void aloca(int **ptr) {
    *ptr = malloc(sizeof(int));
    **ptr = 42;
}

int main(void) {
    int *p = NULL;
    aloca(&p);
    printf("%d\n", *p);  // 42
    free(p);
    return 0;
}
```

## Ponteiros e strings

Strings em C são arrays de `char` terminados em `\0`. Podem ser manipuladas com ponteiros:

```c
char str[] = "Hello";
char *ptr = str;

while (*ptr != '\0') {
    printf("%c", *ptr);
    ptr++;
}
// Imprime: Hello
```

Diferença importante:

```c
char arr[] = "Hello";   // array - pode modificar
char *ptr = "Hello";    // ponteiro para literal - NÃO modifique

arr[0] = 'J';  // OK: arr agora é "Jello"
ptr[0] = 'J';  // ERRADO: comportamento indefinido (literal é read-only)
```

## Ponteiros para funções

Funções também têm endereços. Ponteiros para funções permitem passar funções como argumentos:

```c
#include <stdio.h>

int soma(int a, int b) { return a + b; }
int mult(int a, int b) { return a * b; }

// Função que recebe ponteiro para função
int opera(int x, int y, int (*func)(int, int)) {
    return func(x, y);
}

int main(void) {
    int (*ptr)(int, int);  // declara ponteiro para função

    ptr = soma;
    printf("%d\n", ptr(3, 4));  // 7

    ptr = mult;
    printf("%d\n", ptr(3, 4));  // 12

    // Usando a função opera
    printf("%d\n", opera(5, 3, soma));  // 8
    printf("%d\n", opera(5, 3, mult));  // 15

    return 0;
}
```

## Ponteiros e structs

Acessar membros de struct através de ponteiro:

```c
struct Pessoa {
    char nome[50];
    int idade;
};

struct Pessoa p = {"Maria", 25};
struct Pessoa *ptr = &p;

// Duas formas equivalentes de acessar membros:
printf("%s\n", (*ptr).nome);  // Maria
printf("%s\n", ptr->nome);    // Maria (mais comum)

ptr->idade = 26;  // modifica p.idade
```

O operador `->` é açúcar sintático para `(*ptr).membro`.

## Ponteiro nulo e ponteiro void

```c
// Ponteiro nulo - não aponta para nada
int *ptr = NULL;

if (ptr == NULL) {
    printf("Ponteiro nulo\n");
}

// Sempre verifique antes de desreferenciar
if (ptr != NULL) {
    printf("%d\n", *ptr);
}
```

```c
// Ponteiro void - ponteiro genérico
void *generic;
int x = 10;
float f = 3.14;

generic = &x;
printf("%d\n", *(int *)generic);  // cast necessário

generic = &f;
printf("%.2f\n", *(float *)generic);
```

`void *` é usado em funções genéricas como `malloc`, `qsort`, `bsearch`.

## Alocação dinâmica

Ponteiros são essenciais para alocar memória em tempo de execução:

```c
#include <stdlib.h>

// Alocar um int
int *ptr = malloc(sizeof(int));
if (ptr == NULL) {
    // erro de alocação
    return 1;
}
*ptr = 42;
free(ptr);

// Alocar array de 10 ints
int *arr = malloc(10 * sizeof(int));
for (int i = 0; i < 10; i++) {
    arr[i] = i * 10;
}
free(arr);

// calloc - aloca e zera a memória
int *zeros = calloc(10, sizeof(int));  // 10 ints, todos 0
free(zeros);

// realloc - redimensiona
int *nums = malloc(5 * sizeof(int));
nums = realloc(nums, 10 * sizeof(int));  // agora tem espaço para 10
free(nums);
```

## Erros comuns

### Desreferenciar ponteiro não inicializado

```c
int *ptr;      // aponta para lixo
*ptr = 10;     // ERRO: segmentation fault
```

Solução: sempre inicialize ponteiros:

```c
int *ptr = NULL;  // ou aponte para algo válido
```

### Usar depois de free (use-after-free)

```c
int *ptr = malloc(sizeof(int));
*ptr = 42;
free(ptr);
*ptr = 10;     // ERRO: memória já foi liberada
```

Solução: atribua NULL após free:

```c
free(ptr);
ptr = NULL;
```

### Vazamento de memória (memory leak)

```c
void funcao(void) {
    int *ptr = malloc(sizeof(int));
    // esqueceu de chamar free(ptr)
}  // ptr sai de escopo, memória vazou
```

Solução: sempre libere memória alocada:

```c
void funcao(void) {
    int *ptr = malloc(sizeof(int));
    // usar ptr...
    free(ptr);
}
```

### Retornar ponteiro para variável local

```c
int *funcao(void) {
    int x = 10;
    return &x;  // ERRO: x não existe mais após a função retornar
}
```

Solução: use alocação dinâmica ou variável estática:

```c
int *funcao(void) {
    int *ptr = malloc(sizeof(int));
    *ptr = 10;
    return ptr;  // OK, mas quem chamar deve dar free
}
```

## Exemplo: lista ligada simples

```c
#include <stdio.h>
#include <stdlib.h>

struct Node {
    int valor;
    struct Node *proximo;
};

// Inserir no início
void inserir(struct Node **cabeca, int valor) {
    struct Node *novo = malloc(sizeof(struct Node));
    novo->valor = valor;
    novo->proximo = *cabeca;
    *cabeca = novo;
}

// Imprimir lista
void imprimir(struct Node *cabeca) {
    struct Node *atual = cabeca;
    while (atual != NULL) {
        printf("%d -> ", atual->valor);
        atual = atual->proximo;
    }
    printf("NULL\n");
}

// Liberar lista
void liberar(struct Node *cabeca) {
    struct Node *atual = cabeca;
    while (atual != NULL) {
        struct Node *temp = atual;
        atual = atual->proximo;
        free(temp);
    }
}

int main(void) {
    struct Node *lista = NULL;

    inserir(&lista, 30);
    inserir(&lista, 20);
    inserir(&lista, 10);

    imprimir(lista);  // 10 -> 20 -> 30 -> NULL

    liberar(lista);
    return 0;
}
```

## Resumo

| Conceito | Sintaxe | Significado |
|----------|---------|-------------|
| Declarar ponteiro | `int *ptr;` | Ponteiro para int |
| Endereço de | `&x` | Obtém endereço de x |
| Desreferência | `*ptr` | Valor apontado por ptr |
| Ponteiro nulo | `NULL` | Não aponta para nada |
| Ponteiro para ponteiro | `int **pptr;` | Ponteiro para ponteiro |
| Membro via ponteiro | `ptr->membro` | Acessa membro de struct |
| Aritmética | `ptr + n` | Avança n elementos |

## Links

- [C Reference - Pointers](https://en.cppreference.com/w/c/language/pointer)
- [Beej's Guide - Pointers](https://beej.us/guide/bgc/html/split/pointers.html)
