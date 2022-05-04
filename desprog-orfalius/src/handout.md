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
Antes de começar tome uma água. As contas não vão ser extremamente complicadas. Mas tome uma água, é importante :) 
!!!

Primeiro, precisamos deixar claro que o algoritmo de Karatsuba é um algoritmo de divisão e conquista assim como o merge sort, ou seja, para realizar a multiplicação, o algoritmo:

1. Divide os números que irá multiplicar em números menores.
2. Faz uma árvore recursiva com esses números e vai multiplicando.
3. E no final soma aplica pela última vez o algoritmo de Karatsuba para os últimos valores do topo do árvore.

Então vamos começar pelo primeiro passo, dividindo os números que serão multiplicados. Nesse caso para multiplicar $143721 * 273123$, iremos dividir da seguinte forma:

* x = 143721 e o número pode ser dividido em duas partes: a e b.
* y = 273123 e o número pode ser dividido em duas partes: c e d.

O que pode ser ilustrado na figura abaixo:

![](KaratsubaNum.jpeg)


Como a gente pode ver pela figura acima, separamos os dois termos que estão sendo multiplicados em quatro partes:
- $A = 143$
- $B = 721$
- $C = 273$
- $D = 123$

Logo, tendo os números separados dessa maneira, temos os termos separados para realizarmos as operações necessárias. Além disso, sabemos que:

- x = $A.10^{(n/2)} + B$
- y = $C.10^{(n/2)} + D$

E, com isso, $x.y$ (que é o que queremos calcular, certo?) seria a multiplicação desses dois termos, resultando na **Equação Fundamental de Karatsuba (EFK)** (a gente inventou esse nome, mas isso vai facilitar muito o processo daqui para frente):

- $= A.C.10^{2.(n/2)} + (A.D + B.C).10^{n/2} + B.D$

Eu sei... É bastante coisa, mas dá para ver que não é complicado(....né?). Bom, os próximos passos são um pouco mais complexos e exigem mais atenção, então pega aquela 7 Bello e dá uma boa olhada no que vem por aí. 

??? Checkpoint

Imagine agora o que deveríamos fazer após separar os termos desses números, já que agora temos uma multiplicação entre dois números de 3 dígitos cada um. Porém, como faremos cada uma delas? Pense a respeito disso.

**OBS:** Não precisa fazer nenhuma conta ou raciocínio muito longo ou complexo, apenas pense em como seria.

::: Gabarito

Sabemos que precisaremos calcular $A.C$, $B.D$ e $A.D + B.C$ para aplicar na EFK. Logo, teremos 2 multiplicações entre 3 dígitos cada, a de $A.C$ e $B.D$, correto _meu pequeno gafanhoto_? Sendo assim, ainda precisamos encontrar uma forma de realizar o cálculo de $A.D + B.C$, o que chamaremos de {red}(termo misterioso) sem passos extras desnecessários. Logo, com algumas _matemágicas_:

$= (A + B).(C + D) - A.C - B.D$

$= (A.C + A.D + B.C + B.D) - A.C - A.D$

$= A.D + B.C$

Você deve estar pensando... como raios isso nos ajudou? Bem, é simples, agora nós sabemos que o {red}(termo misterioso) é, na verdade, $(A + B)(C + D) - A.C - B.D$ e, como já estamos realizando o cálculo tanto de $A.C$ quanto de $B.D$, iremos precisar realizar apenas mais uma multiplicação extra, a de $A + B$ por $C + D$.

Portanto, para deixar isso tudo mais visual, teremos as seguintes multiplicações: 


:karatsuba_arv

Agora sim! Tudo pronto, certo? 

Errado! Na verdade o **lema número 1 do Karatsuba** é: Jamais multiplicarás números de 2 ou mais dígitos.

Logo, teremos que realizar mais Karatsubas com todos os 3 ramos gerados e seus subsequentes até atingir uma multiplicação entre 2 números de {red} (1 dígito) e, já que é um algoritmo de divisão e conquista, isso implicará um código que utilizará regressão. 

:::

???

??? Checkpoint

Beleza! Temos a nossa primeira árvore do Karatsuba gerada. Agora, tente fazer, por completo, o ramo da esquerda do exercício acima, chegando no valor final após todas as iterações com o Karatsuba.

!!! Dica

Lembre da grandiosa e famosa **Equação Fundamental de Karatsuba (EFK)**!

$A.C.10^{2.(n/2)} + (A.D + B.C).10^{n/2} + B.D$

!!!

::: Gabarito


:karatsuba_gab


:::

???

Logo, após todos esses exercícios, podemos, finalmente, obter a árvore final dessa conta:


![](karatsuba_gabFINAL.drawio.png)


É... pois é, é bastante coisa. Porém, tudo isso fica muito mais simples quando começamos a codar e deixar o computador fazer! ;)

Qual a complexidade do algoritmo?
---------

No bloco anterior você aprendeu e deduziu a fórmula do Karatsuba. Agora, como seria isso em código?

!!! Aviso
Calma, calma, você não terá que fazer isso do 0. Seguindo os passos do Handout, você irá conseguir :).
!!!

Vimos que o primeiro passo era dividir quem iremos multiplicar em números menores. Sendo assim, precisaremos ir dividindo os números na função até que eles sejam menores que 10, ou seja, possuam um único dígito para multiplicarmos.

Mas antes de fazermos isso, precisaremos determinar quantos dígitos tem os números digitados.

??? Checkpoint

Implemente uma função em C, a `md getSize(long num)` que recebe um número do tipo long e que retorna o número de dígitos.  

::: Gabarito
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
```

:::

???

Após descobrir quantos dígitos tem um número, podemos começar com a estrutura da função que é recursiva. 

??? Checkpoint

Implemente em C, a condição de parada da função `md karatsuba(long X, long Y)` que recebe dois números do tipo long.

!!! Dica
Se você esqueceu a condição de parada, volte ao início desse bloco.
!!!

::: Gabarito
``` c

long karatsuba(long X, long Y)
{
    // Base Case
    if (X < 10 && Y < 10)
        return X * Y;

        ...
}

```

:::

???


Definida a condição de parada, vamos agora acrescentar as recursões. Lembre-se que na árvore de recursão temos 3 recursões a cada execução.
Além disso, também vamos já acrescentar o retorno da função, que é exatamente a equação que deduzimos no bloco anterior.

??? Checkpoint

Implemente em C, as recursões e o retorno do algoritmo. Além disso, já defina o $n$, que será o tamanho do número de dígitos dividido por 2. 

!!! Dica
Pesquisa o que a função `md: fmax` da biblioteca `<math.h>` faz.
!!!

::: Gabarito

Você fez mesmo a função? Se não conseguiu entender, vamos por partes:

**O n é exatamente o maior tamanho dos dígitos dentre os dois números dividido por 2.**

* Por que o maior tamanho dos dígitoa entre os dois números?
Para não perder valores e garantir que estaremos fazendo o karatsuba com todos os dígitos de um número. Dessa forma, imagine que vamos multiplicar 13 e 5200000. Se pegássemos o número de dígitos do 13, que é 2, quando fóssemos dividir os números em partes menores (principalmente o 5200000) teríamos problemas.

* Por que dividido por 2?

Pois na fórmula temos que os dois 10 que aparecem estão elevados a $n/2$. Então isso já ajuda a deixar mais compacto e com menos variáveis na conta final, melhorando a sua vizualização :).


::: Gabarito do gabarito
``` c

long karatsuba(long X, long Y)
{
    // Base Case
    if (X < 10 && Y < 10)
        return X * Y;

    // Recur until base case
    int n = (int)(size / 2.0);

    long ac = karatsuba(a, c);
    long bd = karatsuba(b, d);
    long e = karatsuba(a + b, c + d) - ac - bd;

    // return the equation
    return (long)(pow(10 * 1L, 2 * n) * ac + pow(10 * 1L, n) * e + bd);

}

```

:::

???


A única coisa que falta agora em nossa função é determinar os termos que aparecem na equação, e que irão se utilizar do $n$ determinado anteriormente. Caso você tenha esquecido como descrevemos matematicamente cada um dos termos, volte ao bloco anterior.

??? Checkpoint

Por último, implemente em c as definições dos termos $a$, $b$, $c$ e $d$.

!!! Dica
Sempre que você for dividindo os números pela metade, pode fazer uso da potência de 10...
!!!

::: Gabarito

``` c

long karatsuba(long X, long Y)
{
    // Base Case
    if (X < 10 && Y < 10)
        return X * Y;

    // determine the size of X and Y
    int size = fmax(getSize(X), getSize(Y));

    // Split X and Y
    int n = (int)(size / 2.0);
    long p = (long)pow(10, n);
    long a = (long)(X / (double)p);
    long b = X % p;
    long c = (long)(Y / (double)p);
    long d = Y % p;

    // Recur until base case
    long ac = karatsuba(a, c);
    long bd = karatsuba(b, d);
    long e = karatsuba(a + b, c + d) - ac - bd;

    // return the equation
    return (long)(pow(10 * 1L, 2 * n) * ac + pow(10 * 1L, n) * e + bd);
}

``` 

:::

???

Agora que já temos o código do algoritmo de karatsuba podemos fazer um dos exercícios que você mais gosta : calcular a complexidade do código. Não é muito diferente do que já fizemos em aula. Antes de continuar, uma frase para motivá-lo:

*"As a programmer, never underestimate your ability to come up with ridiculously complex solutions for simple problems." FUCHS, Thomas.* 

![](smart_cat.gif)

E é por isso que é sempre bom nos atentarmos a complexidade das nossas soluções e dos algoritmos que vamos usar. Claro que não apenas isso, mas não deixa de ser uma parte bem importante :)


... Guilherme




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
