O algoritmo de Karatsuba
======

Ideia inicial
---------

Uma das operações mais importantes que aprendemos desde cedo na escola é a multiplicação. E provavelmente o jeito que
você aprendeu no ensino fundamental e a maneira que resolveria uma multiplicação à mão seria dessa forma:

:mult_ingenua

Essa técnica também é conhecida como método ingênuo (ou *"Naive Approach"* em inglês), na qual mutiplicamos cada dígito do multiplicador pelo multiplicando.
E a cada linha da multiplicação nós adicionamos zeros, continuamos multiplicando dígito a dígito, até que no final somamos todas as linhas.

Assim, para o caso anterior, para multiplicar um número de três dígitos por outro número de três dígitos realizamos 9 multiplicações individuais.

??? Checkpoint

Se você tivesse que implementar a multiplicação desse método ingênuo em forma de programação, como faria?

!!! Aviso
Você não precisa implementar o código de fato, apenas pense em sua estrutura e depois confira a resposta.

Observação: não se preocupe com os carries e nem com os zeros. Apenas pense na estrutura do código de maneira geral. 
!!!

::: Gabarito
Você provavelmente deve ter pensando em duas estruturas de loop: uma estrutura de loop externa que vai passando para a próximo dígito conforme os dígitos do multiplicando vão sendo multiplicados, e outra interna que vai passando dígito a dígito conforme os dígitos do multiplicador vão passando pelos
dígitos do multiplicando. 
:::

???


Dessa forma, a estrutura de código seria de maneira geral organizada dessa forma com dois loops:

``` c
for (int i = 0; i < n; i++){
        for (int j = 0; j < n; j++){
            mult_interna[j] = array_num1[i]*array_num2[j]
        }
        
        mult_externa[i] = concatena(row_result[j], 3);
}
    
    return soma(line_results);
```

Suponha que o `md array_num1` guarde em um vetor todos os dígitos do multiplicando, e que o `md array_num2` guarde todos os dígitos do multiplicador.
Ainda, leve em consideração que o `md mult_interna[j]` vá guardando em um vetor o resultado de cada multiplicação individual de uma linha, e que depois
o `md mult_externa` vá guardando o resultado do número inteiro concatenado de todas as linhas. Por fim, considere que existam duas funções, a `md concatena` e a `md soma`.

Um exemplo, considerando a multiplicação mostrada no início do handout seria:

``` c
int array_num1[3] = {1, 2, 3};
int array_num2[3] = {4, 5, 6};

mult_interna[3] = {7, 3, 8} // primeira execução loop interno;
mult_interna[3] = {6, 1, 5} // segunda execução loop interno;
mult_interna[3] = {4, 9, 2} // terceira execução loop interno;

mult_externa[3] = {738, 6150, 49200};

Por fim, soma retornaria: 56088.

```

??? Checkpoint

Nesse método tradicional, se nós multiplicamos dois números de n dígitos, qual seria a sua complexidade?

::: Gabarito

Uma das formas de pensar é: 

Como visto no caso anterior, multiplicando dois números de três dígitos foram necessárias 9 multiplicações individuais. Logo,
ao multiplicar dois números de n dígitos, teremos n² multiplicações individuais. Dessa forma, a complexidade desse método é
de $O(n²)$.

Mas caso você não esteja acreditando ("Tenha fé" HASHIMOTO, Marcelo), resolvendo a complexidade do código mostrado acima temos:

Passo A: Estimar as iterações do loop externo em função de $n$.

``` txt
Quantidade de iterações do loop externo (x):
-------------------------------------------
Contador começa de 0 e aumenta em 1 enquanto for menor que n.

Depois de x-1 iterações, a condição ainda vale.
0 + 1(x-1) < n
x - 1 < n
x < n + 1

Depois de x iterações, a condição não vale.
0 + x >= n
x >= n 

n <= x < n+1
``` 

Passo B: Estimar as iterações do loop interno em função de $n$.

Calculamos apenas o limitante superior, pois o inferior nunca é necessário.

``` txt
Quantidade de iterações do loop interno (y):
-------------------------------------------
Contador começa de 0 e aumenta em 1 enquanto for menor que n.

Depois de y-1 iterações, a condição ainda vale.
0 + 1(y-1) < n
y - 1 < n
y  < n + 1

y < n+1

``` 

Passo C: Listar o valor de  em cada iteração do loop externo.

O número de valores é **sempre** $x$ e o último valor é **sempre** dado pela mesma fórmula usada no Passo A.

``` txt
Valor de i ao longo do loop externo:
-----------------------------------
i = 0, 1, 2, 3, 4, ..., 0 + 1(x-1)
``` 

Passo D: Juntar os Passos B e C. 

``` txt
Limitante para as iterações de todas as execuções do loop interno:
-----------------------------------------------------------------

0, 1, 2, 3, 4, ..., 0 + 1(y-1)


Numerador é soma de PA com:
- primeiro elemento 0;
- último elemento y - 1 ;
- número de elementos x.

 = ((0 + y-1)* x)/2
 = (x*y - x)/2

Como y < n+1 e x < n+1:

= (n^2 + 2n + 1 - n - 1) / 2
= (n^2 + n)/2

``` 

Logo a complexidade seria de $O(n²)$.

???

E como vimos em aula, uma complexidade quadrática é algo que queremos evitar, sendo pior que a complexidade linear, logarítmica, e constante.

Isso porque, imagine alguns dos processos que utilizam da multiplicação, principalmente com números grandes. Ou melhor imagine multiplicar milhões de dígitos por
milhões de dígitos. É isso que acontece em muitas aplicações de processamento de sinais que são correlação, convolução, análise de frequência, processamento de imagem, etc.
Além disso, a eficiência de multiplicações é uma base para implementação de moduladores, criptossistemas e até mesmo para a ULA (Unidade Lógica Aritmética) que vimos na matéria de Elementos de Sistemas... Você lembra, né? 

Pense que nos criptossistemas, por exemplo, a aritmética modular é a operação central na sua grande maioria. E muitos dos sistemas criptográficos requerem multiplicações modulares para gerar chaves privadas, fazendo o uso de exponenciação modular de grandes números para criptografar dados, o que é um processo lento por si só devido a repetição de muitas multiplicações. Agora imagine se a eficiência da operação é baixa... Complicado, não?

É por isso que muitas companias como a Intel utilizam o algoritmo de Karatsuba, pois ele foi desenvolvido para aumentar a eficiência e simplificar a multiplicação - principalmente para números grandes. Um breve *background* do algoritmo é que ele foi desenvolvido em 1960 por Anatoly Karatsuba, um matemático que nasceu na União Soviética (<a href="https://www.youtube.com/watch?v=U06jlgpMtQs"> Importante </a>) e que morreu em 2008, tendo trabalhado principalmente nos campos da teoria analítica de número e nas séries de Dirichlet (não se preocupe com o que é, só por ter um nome difícil já é legal).


Como o algoritmo funciona?
---------

Já falamos que o algoritmo de Karatsuba serve para simplificar a multiplicação. Mas como?

!!! Aviso
Antes de começar tome uma água. As contas não vão ser extremamente complicadas. Mas tome uma água. 
!!!

Primeiro, precisamos deixar claro que o algoritmo de Karatsuba é um algoritmo de divisão e conquista assim como o merge sort, ou seja, para realizar a multiplicação, o algoritmo:

1. Divide os números que irá multiplicar em números menores.
2. Faz uma árvore recursiva com esses números e vai multiplicando.
3. E no final soma aplica pela última vez o algoritmo de Karatsuba para os últimos valores do topo do árvore.

Então vamos começar pelo primeiro passo, dividindo os números que serão multiplicados. Nesse caso para multiplicar $143721 * 273123$, iremos dividir da seguinte forma:

* x = 143721 e o número pode ser dividido em duas partes: a e b.
* y = 273123 e o número pode ser dividido em duas partes: c e d.


``` c
#include <stdio.h>
#include <math.h>

// Get size of the numbers
int getSize(long num)
{
    int count = 0;
    while (num > 0)
    {
        count++;
        num /= 10;
    }
    return count;
}

long karatsuba(long X, long Y)
{
    // Base Case
    if (X < 10 && Y < 10)
        return X * Y;

    // determine the size of X and Y
    int size = fmax(getSize(X), getSize(Y));

    // Split X and Y
    int n = (int)ceil(size / 2.0);
    long p = (long)pow(10, n);
    long a = (long)floor(X / (double)p);
    long b = X % p;
    long c = (long)floor(Y / (double)p);
    long d = Y % p;

    // Recur until base case
    long ac = karatsuba(a, c);
    long bd = karatsuba(b, d);
    long e = karatsuba(a + b, c + d) - ac - bd;

    // return the equation
    return (long)(pow(10 * 1L, 2 * n) * ac + pow(10 * 1L, n) * e + bd);
}

``` 



<!-- 
Você também pode criar

1. listas;

2. ordenadas,

assim como

* listas;

* não-ordenadas

e imagens. Lembre que todas as imagens devem estar em uma subpasta *img*.

![](logo.png)

Para tabelas, usa-se a [notação do
MultiMarkdown](https://fletcher.github.io/MultiMarkdown-6/syntax/tables.html),
que é muito flexível. Vale a pena abrir esse link para saber todas as
possibilidades.

| coluna a | coluna b |
|----------|----------|
| 1        | 2        |

Ao longo de um texto, você pode usar *itálico*, **negrito**, {red}(vermelho) e
[[tecla]]. Também pode usar uma equação LaTeX: $f(n) \leq g(n)$. Se for muito
grande, você pode isolá-la em um parágrafo.

$$\lim_{n \rightarrow \infty} \frac{f(n)}{g(n)} \leq 1$$

Para inserir uma animação, use `md :` seguido do nome de uma pasta onde as
imagens estão. Essa pasta também deve estar em *img*.

:bubble

Você também pode inserir código, inclusive especificando a linguagem.

``` py
def f():
    print('hello world')
```

``` c
void f() {
    printf("hello world\n");
}
```

Se não especificar nenhuma, o código fica com colorização de terminal.

```
hello world
```


!!! Aviso
Este é um exemplo de aviso, entre `md !!!`.
!!!


??? Exercício

Este é um exemplo de exercício, entre `md ???`.

::: Gabarito
Este é um exemplo de gabarito, entre `md :::`.
:::

??? -->
