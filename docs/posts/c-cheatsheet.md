---
date: 2025-12-10
categories:
  - Programação
  - C
---

# C: referência rápida

Fundamentos da linguagem C para iniciantes.

<!-- more -->

## Compilação

```bash
# Compilar
gcc programa.c -o programa

# Compilar com warnings
gcc -Wall -Wextra programa.c -o programa

# Compilar para debug
gcc -g programa.c -o programa

# Executar
./programa
```

## Estrutura básica

```c
#include <stdio.h>

int main(void) {
    printf("Hello, World!\n");
    return 0;
}
```

## Tipos de dados

```c
// Inteiros
char c = 'A';           // 1 byte (-128 a 127)
short s = 32767;        // 2 bytes
int i = 2147483647;     // 4 bytes
long l = 9223372036854775807L;  // 8 bytes

// Sem sinal
unsigned char uc = 255;
unsigned int ui = 4294967295;

// Decimais
float f = 3.14f;        // 4 bytes, 6-7 dígitos
double d = 3.14159265;  // 8 bytes, 15-16 dígitos

// Tamanho dos tipos
printf("int: %zu bytes\n", sizeof(int));
printf("double: %zu bytes\n", sizeof(double));
```

## Variáveis e constantes

```c
// Declaração
int idade;
int x, y, z;

// Inicialização
int contador = 0;
float preco = 19.99f;

// Constantes
const int MAX = 100;
#define PI 3.14159
```

## Operadores

```c
// Aritméticos
int soma = a + b;
int sub = a - b;
int mult = a * b;
int div = a / b;      // divisão inteira se ambos int
int resto = a % b;    // módulo

// Incremento/Decremento
i++;    // i = i + 1
i--;    // i = i - 1
++i;    // incrementa antes de usar
--i;    // decrementa antes de usar

// Atribuição composta
x += 5;   // x = x + 5
x -= 3;   // x = x - 3
x *= 2;   // x = x * 2
x /= 4;   // x = x / 4

// Comparação
a == b    // igual
a != b    // diferente
a > b     // maior
a < b     // menor
a >= b    // maior ou igual
a <= b    // menor ou igual

// Lógicos
a && b    // AND
a || b    // OR
!a        // NOT

// Bitwise
a & b     // AND bit a bit
a | b     // OR bit a bit
a ^ b     // XOR
~a        // NOT (complemento)
a << 2    // shift left
a >> 2    // shift right
```

## Entrada e saída

```c
#include <stdio.h>

// Saída
printf("Texto simples\n");
printf("Inteiro: %d\n", 42);
printf("Float: %.2f\n", 3.14159);
printf("Char: %c\n", 'A');
printf("String: %s\n", "texto");
printf("Múltiplos: %d e %s\n", 10, "dez");

// Entrada
int idade;
printf("Digite sua idade: ");
scanf("%d", &idade);  // & pega o endereço

char nome[50];
printf("Digite seu nome: ");
scanf("%s", nome);    // sem & para arrays

// Ler linha inteira (com espaços)
char linha[100];
fgets(linha, sizeof(linha), stdin);
```

### Especificadores de formato

| Formato | Tipo |
|---------|------|
| `%d` | int |
| `%ld` | long |
| `%u` | unsigned int |
| `%f` | float/double |
| `%.2f` | float com 2 casas |
| `%c` | char |
| `%s` | string |
| `%p` | ponteiro |
| `%x` | hexadecimal |
| `%%` | literal % |

## Condicionais

```c
// if-else
if (idade >= 18) {
    printf("Maior de idade\n");
} else if (idade >= 12) {
    printf("Adolescente\n");
} else {
    printf("Criança\n");
}

// Ternário
int max = (a > b) ? a : b;

// switch
switch (opcao) {
    case 1:
        printf("Opção 1\n");
        break;
    case 2:
        printf("Opção 2\n");
        break;
    default:
        printf("Opção inválida\n");
}
```

## Loops

```c
// for
for (int i = 0; i < 10; i++) {
    printf("%d\n", i);
}

// while
int i = 0;
while (i < 10) {
    printf("%d\n", i);
    i++;
}

// do-while (executa pelo menos uma vez)
int j = 0;
do {
    printf("%d\n", j);
    j++;
} while (j < 10);

// Controle de fluxo
for (int i = 0; i < 10; i++) {
    if (i == 3) continue;  // pula iteração
    if (i == 7) break;     // sai do loop
    printf("%d\n", i);
}
```

## Arrays

```c
// Declaração e inicialização
int numeros[5];                      // não inicializado
int valores[5] = {1, 2, 3, 4, 5};    // inicializado
int zeros[5] = {0};                  // todos zeros
int parcial[5] = {1, 2};             // resto é zero

// Tamanho inferido
int arr[] = {1, 2, 3, 4, 5};  // tamanho = 5

// Acesso
arr[0] = 10;           // primeiro elemento
arr[4] = 50;           // último elemento
int tamanho = sizeof(arr) / sizeof(arr[0]);

// Percorrer
for (int i = 0; i < 5; i++) {
    printf("%d\n", arr[i]);
}

// Array multidimensional
int matriz[3][3] = {
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9}
};
printf("%d\n", matriz[1][2]);  // 6
```

## Strings

```c
#include <string.h>

// Strings são arrays de char terminados em '\0'
char nome[20] = "João";
char outro[] = "Maria";  // tamanho = 6 (5 + '\0')

// Funções de string
strlen(str);              // tamanho (sem '\0')
strcpy(dest, src);        // copia src para dest
strncpy(dest, src, n);    // copia até n chars
strcat(dest, src);        // concatena
strncat(dest, src, n);    // concatena até n
strcmp(s1, s2);           // compara (0 se iguais)
strncmp(s1, s2, n);       // compara n chars
strchr(str, c);           // busca char
strstr(str, substr);      // busca substring

// Exemplo
char saudacao[50];
strcpy(saudacao, "Olá, ");
strcat(saudacao, nome);
strcat(saudacao, "!");
printf("%s\n", saudacao);  // "Olá, João!"
```

## Funções

```c
// Declaração (protótipo)
int soma(int a, int b);
void imprime(char *msg);

// Definição
int soma(int a, int b) {
    return a + b;
}

void imprime(char *msg) {
    printf("%s\n", msg);
}

// Chamada
int resultado = soma(3, 5);
imprime("Hello");

// Passagem por valor (cópia)
void dobra(int x) {
    x = x * 2;  // não afeta o original
}

// Passagem por referência (ponteiro)
void dobra_ref(int *x) {
    *x = *x * 2;  // afeta o original
}

int num = 10;
dobra(num);       // num ainda é 10
dobra_ref(&num);  // num agora é 20
```

## Ponteiros

```c
int x = 10;
int *ptr;       // declara ponteiro para int

ptr = &x;       // ptr recebe endereço de x

printf("%d\n", x);      // 10 (valor)
printf("%p\n", &x);     // endereço de x
printf("%p\n", ptr);    // mesmo endereço
printf("%d\n", *ptr);   // 10 (valor apontado)

*ptr = 20;      // altera x através do ponteiro
printf("%d\n", x);  // 20

// Ponteiro e arrays
int arr[] = {1, 2, 3, 4, 5};
int *p = arr;   // aponta para primeiro elemento

printf("%d\n", *p);       // 1
printf("%d\n", *(p + 1)); // 2
printf("%d\n", p[2]);     // 3 (notação de array)

// Aritmética de ponteiros
p++;            // avança para próximo int
printf("%d\n", *p);  // 2

// Ponteiro nulo
int *nulo = NULL;
if (nulo == NULL) {
    printf("Ponteiro nulo\n");
}
```

## Structs

```c
// Definição
struct Pessoa {
    char nome[50];
    int idade;
    float altura;
};

// Uso
struct Pessoa p1;
strcpy(p1.nome, "João");
p1.idade = 30;
p1.altura = 1.75;

// Inicialização direta
struct Pessoa p2 = {"Maria", 25, 1.65};

// Com typedef
typedef struct {
    int x;
    int y;
} Ponto;

Ponto origem = {0, 0};
printf("(%d, %d)\n", origem.x, origem.y);

// Ponteiro para struct
struct Pessoa *ptr = &p1;
printf("%s\n", ptr->nome);    // operador ->
printf("%d\n", (*ptr).idade); // equivalente
```

## Alocação dinâmica

```c
#include <stdlib.h>

// malloc - aloca memória (não inicializa)
int *ptr = (int *)malloc(sizeof(int));
if (ptr == NULL) {
    printf("Erro de alocação\n");
    return 1;
}
*ptr = 42;

// calloc - aloca e inicializa com zeros
int *arr = (int *)calloc(10, sizeof(int));

// realloc - redimensiona
arr = (int *)realloc(arr, 20 * sizeof(int));

// free - libera memória
free(ptr);
free(arr);
ptr = NULL;  // boa prática
arr = NULL;

// Array dinâmico
int n = 5;
int *numeros = (int *)malloc(n * sizeof(int));
for (int i = 0; i < n; i++) {
    numeros[i] = i * 10;
}
free(numeros);
```

## Arquivos

```c
#include <stdio.h>

// Abrir arquivo
FILE *fp = fopen("dados.txt", "r");  // leitura
FILE *fp = fopen("dados.txt", "w");  // escrita (sobrescreve)
FILE *fp = fopen("dados.txt", "a");  // append
FILE *fp = fopen("dados.txt", "r+"); // leitura e escrita

if (fp == NULL) {
    printf("Erro ao abrir arquivo\n");
    return 1;
}

// Escrever
fprintf(fp, "Linha %d\n", 1);
fputs("Outra linha\n", fp);
fputc('A', fp);

// Ler
char linha[100];
while (fgets(linha, sizeof(linha), fp) != NULL) {
    printf("%s", linha);
}

int num;
fscanf(fp, "%d", &num);

int c;
while ((c = fgetc(fp)) != EOF) {
    putchar(c);
}

// Fechar
fclose(fp);
```

## Pré-processador

```c
// Include
#include <stdio.h>    // biblioteca padrão
#include "meuheader.h" // arquivo local

// Define
#define PI 3.14159
#define MAX(a, b) ((a) > (b) ? (a) : (b))
#define QUADRADO(x) ((x) * (x))

// Condicional
#define DEBUG 1

#ifdef DEBUG
    printf("Debug: x = %d\n", x);
#endif

#ifndef HEADER_H
#define HEADER_H
// conteúdo do header
#endif
```

## Erros comuns

```c
// Array fora dos limites
int arr[5];
arr[5] = 10;  // índice 5 não existe!

// Usar ponteiro não inicializado
int *ptr;
*ptr = 10;  // segfault!

// Esquecer de liberar memória
int *p = malloc(sizeof(int));
// ... usar p ...
// esqueceu free(p) = memory leak

// Usar depois de free
free(ptr);
*ptr = 10;  // comportamento indefinido!

// Buffer overflow em strings
char nome[5];
strcpy(nome, "João da Silva");  // estoura o buffer!

// Usar strncpy
strncpy(nome, "João da Silva", sizeof(nome) - 1);
nome[sizeof(nome) - 1] = '\0';
```

## Links

- [C Reference](https://en.cppreference.com/w/c)
- [Learn C](https://www.learn-c.org/)
- [Beej's Guide to C](https://beej.us/guide/bgc/)
