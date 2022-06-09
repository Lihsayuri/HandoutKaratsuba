O algoritmo de Karatsuba
======

Introdução (expositiva)
---------


Muitos dos processos computacionais utilizam multiplicação, principalmente com números **grandes** - e grandes mesmo, na casa de bilhões. É isso que acontece em muitas aplicações de processamento de sinais que são correlação, convolução, análise de frequência, processamento de imagem, etc. Além disso, a eficiência de multiplicações é uma base para implementação de moduladores, criptossistemas e até mesmo para a ULA (Unidade Lógica Aritmética) que vimos na matéria de Elementos de Sistemas... Você lembra, né? **NÉ?** (Diga sim, deixe o Corsi feliz :) ).

Pense que nos criptossistemas, por exemplo, a aritmética modular é a operação central na sua grande maioria. E muitos dos sistemas criptográficos requerem multiplicações modulares para gerar chaves privadas, fazendo o uso de exponenciação modular de grandes números para criptografar dados, o que é um processo lento por si só devido a repetição de muitas multiplicações. Agora imagine se a eficiência da operação é baixa... Complicado, não? Hoje vamos ver uma técnica que foi utilizada por muitas empresas, como a Intel, e que é mais eficiente para realizar multiplicações 
-**principalmente para números grandes** (a essa altura você já deve ter entendido isso).


Ideia inicial
---------

Uma das operações mais importantes que aprendemos desde cedo na escola é a multiplicação. E provavelmente o jeito que
você aprendeu no ensino fundamental e a maneira que resolveria uma multiplicação à mão seria dessa forma:

:mult_ingenua

Essa técnica também é conhecida como método ingênuo (ou *"Naive Approach"* em inglês), na qual mutiplicamos cada dígito do multiplicador pelo multiplicando. 
E a cada linha da multiplicação nós adicionamos zeros, continuamos multiplicando dígito a dígito, até que no final somamos todas as linhas.

Assim, para o caso anterior, para multiplicar um número de três dígitos por outro número de três dígitos, realizamos 9 multiplicações individuais.



??? Checkpoint

Se você tivesse que implementar a multiplicação desse método ingênuo em forma de programação com **vetores** representando cada um dos dois números, como faria?

- Lembra de Elementos de Sistemas? Para implementar uma multiplicação simples (de um único dígito), em Assembly nós realizávamos várias somas. Agora pense que você possui a multiplicação de um único dígito disponível, mas precisa mutiplicar vários dígitos... 


!!! Aviso
Você não precisa implementar o código de fato, apenas pense em sua estrutura e depois confira a resposta.

Observação: não se preocupe com os *carries* (famosos "sobe 1") e nem com os zeros. Apenas pense na estrutura do código de maneira geral. 
!!!

::: Gabarito
Você provavelmente deve ter pensando em duas estruturas de loop: uma estrutura de loop externa que vai passando para a próximo dígito conforme os dígitos do multiplicando vão sendo multiplicados, e outra interna que vai passando dígito a dígito pelos dígitos do multiplicador. E isso feito com um vetor, assim, se fôssemos multiplicar 123 por 456, por exemplo, teríamos a = {1, 2, 3} e b = {4, 5, 6} e o primeiro loop passaria multiplicando 4, 5, 6 por 3.
:::

???


Dessa forma, como descrito anteriormente, a estrutura de código seria de maneira geral organizada como vetores e com dois loops, dessa forma:

``` c

array_num1 = {1, 2, 3}
array_num2 = {4, 5, 6}

for (int i = 0; i < n; i++){
        for (int j = 0; j < n; j++){
            mult_interna[j] = array_num1[i]*array_num2[j]
        }
        
        mult_externa[i] = concatena(mult_interna[j], 3);
}
    
    return soma(line_results);
```

Suponha que o `md array_num1` guarde em um vetor todos os dígitos do multiplicando, e que o `md array_num2` guarde todos os dígitos do multiplicador.
Ainda, leve em consideração que o `md mult_interna[j]` vá guardando em um vetor o resultado de cada multiplicação individual de uma linha, e que depois
o `md mult_externa` vá guardando o resultado do número inteiro concatenado de todas as linhas. Por fim, considere que existam duas funções, a `md concatena` e a `md soma`.

Um exemplo com saídas, considerando a multiplicação mostrada no início do handout, seria assim:

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

Nesse método tradicional, se nós multiplicamos dois números de $n$ dígitos, qual seria a sua complexidade?

!!! Aviso
Não precisa resolver a conta, reflita sobre o código que você acabou de pensar. Dois loops... (ok, chega de dica!)
!!!

::: Gabarito

Uma das formas de pensar é: 

Como visto no caso anterior, multiplicando dois números de três dígitos foram necessárias 9 multiplicações individuais. Logo,
ao multiplicar dois números de n dígitos, teremos n² multiplicações individuais. Dessa forma, a complexidade desse método é
de $O(n²)$.

::: Resolução opcional (depois de terminar o Handout, se quiser ver, está aqui a resolução matemática detalhada)

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

Como vimos em aula, uma complexidade quadrática é algo que queremos evitar, sendo pior que a complexidade linear, logarítmica, e constante.
E é aí que entra o algoritmo de Karatsuba. Ele não é o melhor algoritmo de todos para diminuir a complexidade de multiplicações, já que existem outros algoritmos como o de Toom-Cook e o Schönhage–Strassen, que são mais rápidos (<a href="https://en.wikipedia.org/wiki/Karatsuba_algorithm"> Karatsuba Wikipedia </a>). Mas né, vamos dar um desconto, foi desenvolvido em 1960 por Anatoly Karatsuba, um matemático que nasceu na União Soviética (<a href="https://www.youtube.com/watch?v=U06jlgpMtQs"> Importante </a>) e que morreu em 2008


Como o algoritmo funciona?
---------

Já falamos que o algoritmo de Karatsuba serve para simplificar a multiplicação. Mas como?

!!! Aviso
Antes de começar tome uma água. As contas não vão ser extremamente complicadas. Mas tome uma água, é importante :) 
!!!

Primeiro, precisamos deixar claro que o algoritmo de Karatsuba é um algoritmo de divisão e conquista assim como o *merge sort*, ou seja, para realizar a multiplicação, o algoritmo divide um problema maior em problemas menores até atingir um problema de solução mais simples possível.

![](divide_conquer.jpg)

??? Checkpoint

Certo, então pensando agora em uma multiplicação entre dois números de 6 dígitos cada, como podemos simplificar e tornar todas essas contas em multiplicações de 1 dígito só? Ou seja, ao invés de multiplicarmos 6 dígitos por 6 dígitos e termos diversas contas para fazer, como chegaríamos em um produto de 1 dígito por 1 dígito?

**OBS:** Não tente pensar em nada muito complexo, a resposta pode ser mais simples do que imagina. Tente escrever um passo a passo do que você faria para resolver o problema.

::: Gabarito

1. Primeiro, dividimos os números que iremos multiplicar em números menores;

2. Depois, realizamos mais divisões em números ainda menores;

3. Por fim, faremos isso até atingirmos uma multiplicação de menor simplicidade possível, ou seja, uma multiplicação entre dois números de um único dígito.

:::

???

Então, para poder visualizar como isso ocorre de fato, vamos começar pelo começo, dividindo os números que serão multiplicados. Nesse caso para multiplicar $143721.273123$, iremos dividir da seguinte forma:

* X = 143721 e o número pode ser dividido em duas partes: a e b.
* Y = 273123 e o número pode ser dividido em duas partes: c e d.

Para ficar mais claro visualmente:

![](KaratsubaNum.jpeg)

??? Checkpoint

Certo, agora já sabemos que cada número deve ser dividido em 2 partes com 3 dígitos. Logo, pense na lógica de como essa divisão é feita para números de quaisquer quantidades de algoritmos, não apenas para esse caso exclusivo de multiplicação entre 2 números de 6 dígitos.

::: Gabarito

Você deve ter pensado em sempre dividir esse número pela metade do número total de dígitos, e você está certo!
Porém, e se tivermos uma quantidade ímpar de dígitos?

::: Mais que um gabarito

Bom, não tem muita diferença, a realidade é que tanto faz. Por exemplo, se iremos dividir o número 12345 em duas parte para o Karatsuba, tanto faz usarmos 123 e 45 como as duas partes ou 12 e 345. Mais para frente você vai entender o porquê disso.

:::

???

Bom, então usando o exemplo de cima, podemos separar os dois termos que estão sendo multiplicados em quatro partes:
- $A = 143$
- $B = 721$
- $C = 273$
- $D = 123$

??? Checkpoint

Certo, sendo assim, se obtivermos essa separação, teremos os termos necessários para seguir com o método de Karatsuba.  Mas então como chegamos em X e Y a partir de A,B,C e D? Que contas são necessárias fazer? 

!!! Não se esqueça

Sabemos que o X é o primeiro número (143721) e Y o segundo (273123).

Pense em uma fórmula geral e lembre-se do método que estamos utilzando: "**DIVIDE** and Conquer".

!!! 

::: Gabarito

Tendo $n$ como o número de dígitos, sabemos que os números $X$ e $Y$ podem ser representados da seguinte maneira:

- x = $A.10^{(n/2)} + B$
- y = $C.10^{(n/2)} + D$

:::

???

E, com isso, $X.Y$ (que é o que queremos calcular, certo?) seria a multiplicação desses dois termos que foram representados acima, resultando na **Equação Fundamental de Karatsuba (EFK)** (a gente inventou esse nome, mas isso vai facilitar muito o processo daqui para frente).

??? Checkpoint

Tranquilo então, agora que temos todos os termos podemos montar a EFK. Agora, realize a multiplicação de X por Y (em função de A,B,C e D) com o que foi visto acima. O que teremos?

::: Gabarito

A resposta é realizar simplesmente uma distributiva. 
Sabemos que:

$EFK = X.Y$

$EFK = (A.10^{(n/2)} + B).(C.10^{(n/2)} + D)$

$EFK = AC.10^{2.(n/2)} + (AD + BC).10^{n/2} + BD$

:::

???

No entanto, uma outra reflexão é muito importante para continuarmos. 

Lembra que o nosso objetivo desde o ínicio é reduzir o número de multiplicações realizadas para chegarmos ao valor final da multiplicação, certo? Bom, como os próximos passos são um pouco mais complexos e exigem mais atenção, já pega aquela 7 Bello e se liga no que vem por aí. 

??? Checkpoint

Agora, tendo em vista que queremos reduzir o número de multiplicações necessárias, realize a distributiva $(A + B).(C + D)$ e, após isso, tente chegar no termo (AD + BC) a partir dela. 

!!! Dica

Somas ou subtrações extras não alteram a complexidade do algoritmo, já que são todas O(n) e, além disso, menores que a complexidade de uma multiplicação.

!!!

::: Gabarito

Sabemos que precisaremos calcular $AC$, $BD$ e $AD + BC$ para aplicar na EFK. Logo, teremos de cara 2 multiplicações entre 3 dígitos cada, a de $AC$ e $BD$, correto _meu pequeno gafanhoto_? Sendo assim, ainda precisamos encontrar uma forma de realizar o cálculo de $AD + BC$, o que chamaremos de {red}(termo misterioso) sem passos extras desnecessários. Logo, a partir da dica acima e com algumas _matemágicas_ , ou seja, retirando os termos desnecessários por meio de subtrações (já que, como dito acima, as somas não alteram a complexidade do algoritmo):

$= (A + B).(C + D) - AC - BD$

$= (AC + AD + BC + BD) - AC - BD$

$= AD + BC$

Ou seja, estaremos trocando uma multiplicação por somas. Ao invés de realizarmos um Karatsuba com $AD$ e $BC$, estaremos fazendo um com $A+B$ e outro com $C+D$ e não esquecendo de subtrair **do resultado** $AC$ e $BD$ - que já terão sido calculados.

Portanto, para deixar isso tudo mais visual, teremos as seguintes multiplicações: 

:karatsuba_arv

:::

???

Agora sim! Tudo pronto, certo? 

Errado! Lembre da ideia principal do método de divisão e conquista, precisamos chegar às multiplicações mais simples possíveis.

Logo, teremos que realizar mais Karatsubas com todos os 3 ramos gerados e seus subsequentes até atingir uma multiplicação entre 2 números de 
{red}(1 dígito) e, já que é um algoritmo de divisão e conquista, isso implicará um código que utilizará regressão. 


??? Checkpoint

![](karatsuba_arv/karatsuba_arv_04.png)

Beleza! Acima nós temos a nossa primeira árvore do Karatsuba gerada, a qual foi feita no checkpoint anterior, lembra?. Agora, tente fazer, por completo, o ramo da esquerda do exercício acima, chegando no valor final após todas as iterações com o Karatsuba.

!!! Dica

Lembre da grandiosa e famosa **Equação Fundamental de Karatsuba (EFK)**!

$AC.10^{2.(n/2)} + (AD + BC).10^{n/2} + BD$

!!!

::: Gabarito


:karatsuba_gab


:::

???

Logo, após todos esses exercícios, podemos, finalmente, obter a árvore final dessa conta:


![](karatsuba_gabFINAL.drawio.png)


É... pois é, é bastante coisa. Porém, tudo isso fica muito mais simples quando começamos a codar e deixar o computador fazer! ;)

Para facilitar a sua vida e deixar mais visual - *programicamente* falando, se é que você me entende -  aqui está o código do Karatsuba implementado em C - que você provavelmente vai precisar para o próximo bloco...

Se ficou curioso para entender os detalhes do código, quando terminar o Handout faça a sessão extra. 


``` c
#include <math.h>
#include <stdio.h>

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
    // Caso base
    if (X < 10 && Y < 10)
        return X * Y;

    // Determina o tamanho de X e Y e pega o maior
    int size = fmax(getSize(X), getSize(Y));

    // Separa o X do Y e define quem é o a, o b, o c e o d
    int n = (int)(size / 2.0);
    long p = (long)pow(10, n);
    long a = (long)(X / (double)p);
    long b = X % p;
    long c = (long)(Y / (double)p);
    long d = Y % p;

    // Faz as recursões até o caso base como foi descrito na sessão anterior
    long ac = karatsuba(a, c); 
    long bd = karatsuba(b, d);
    long e = karatsuba(a + b, c + d) - ac - bd;

    // Retorna a equação
    return (long)(pow(10, 2 * n) * ac + pow(10, n) * e + bd);
}

``` 


Calculando a Complexidade
------


Já que temos o código do algoritmo de Karatsuba e você já entendeu a lógica de como ele funciona, podemos fazer um dos exercícios que você mais gosta : calcular a complexidade do código. Não é muito diferente do que já fizemos em aula. Antes de continuar, uma frase para motivá-lo:

*"As a programmer, never underestimate your ability to come up with ridiculously complex solutions for simple problems." FUCHS, Thomas.* 

![](smart_cat.gif)

E é por isso que é sempre bom nos atentarmos a complexidade das nossas soluções e dos algoritmos que vamos usar. Claro que não apenas isso, mas não deixa de ser uma parte bem importante...

Agora, você deve estar se perguntando: "Ok ok, entendi que simplifica as multiplicações e que a complexidade vai melhorar, mas quanto? Ela cai bastante?". É isso que vamos responder aqui, ou melhor, você vai.

Como já vimos em aulas passadas, com algoritmos com recursão, é preciso fazer a árvore de recursões para entender a complexidade. 

??? Checkpoint

Vamos começar a pensar na árvore de recursão. Pensando no código e quantas recursões temos, quantos ramos temos em cada iteração? **LEMBRE-SE :** quando resolvíamos complexidades com recursões nós sempre pensávamos em como a recursão vai acontecendo e também na sua condição de parada...  
 
::: Gabarito

Como visto anteriormente, temos 3 recursões e a condição de parada (sim, os vermelhos da árvore, os vermelhos...). Portanto nossa árvore de recursões terá 4 ramos em cada iteração.

:::
???

??? Checkpoint

Agora, o próximo passo é pensar nas iterações! Pensando em como o algoritmo (baseado em *divisão e conquista*) trabalha com os números que recebe, escreva como fica a chamada das recursões depois da primeira chamada **f(n)**. Lembre-se também que além da recursão, temos a soma das iterações, que aparecerá como um ramo a mais na árvore.

::: Gabarito

Como em cada iteração o algoritmo divide o tamanho do número que ele tem por 2, teremos 3 ramos de **f(n/2)**, e um ramo que é a volta da recursão (e faz a soma da etapa), como você viu anteriormente.

:::
???

??? Checkpoint

Por fim, tente fazer a árvore de recursão inteira agora que você sabe como cada iteração será!

::: Gabarito

![](arvRecursao.png)

Em cada degrau da árvore, temos três ramos que indicam as recursões, onde cada uma é **f(*n*/2)**, e um ramo que representa o que acontece na *volta*, ou seja, a conta que "junta" tudo. Nesse ramo da volta, a complexidade é o próprio ***n*/2**.

Para ilustrar, veja o degrau de *f(n/2)* e seus ramos. Cada ramo  é *f(n/4)*, e o ramo de volta é o próprio *n/2*.

:::

???

Sabendo a árvore de recursão, podemos usá-la para calcular a complexidade inteira. Prepare-se que vamos usar um pouco daquela *matemágica*!

??? Checkpoint

Agora vamos calcular o algoritmo de Karatsuba passo a passo. **Passo a passo**, não tente apressar!

!!! Dica
Revise o material no <a href=https://ensino.hashi.pro.br/desprog/resumo/analise/caixa.html> site da disciplina</a> em caso de dúvidas.
!!! 
**Passo 1:**

Vamos começar calculando a altura da árvore (denominaremos como x). Qual é a relação de $n$ com a altura?

::: Gabarito Passo 1

Primeiro é importante identificar a altura de nossa árvore. Como $n$ vai reduzindo sendo dividido por 2, sabemos que a altura será **$log_2(n)$**.

* [[x = $log_2n$]]

:::

**Passo 2:**

Como vimos ao longo da matéria, para calcular a recursão precisamos saber qual é a complexidade do ramo referente ao *retorno da recursão* e dos outros ramos da própria recurssão, mas como o tempo está curto vamos calcular apenas a primeira (os famosos {red}(vermelhos)) que é quem de fato vai ditar a complexidade. Dessa forma, tente calcular isso agora, qual o padrão que o retorno das recursões seguem?

::: Gabarito Passo 2

Assim, primeiro precisamos identificar o padrão do retorno da recursão (os passos em {red}(vermelho) na figura anterior). Não podemos esquecer que, para não deixar a imagem muito poluída, não foram mostradas as recursões dos dois ramos do meio, mas ainda temos ramos {red}("vermelhos") neles. Essa próxima imagem mostra a lógica apenas desses ramos:

![](arvRecursao2.png)

Começamos com 1 vez n ([[{red}(n)]]), depois temos 3 vezes n/2 ([[{red}(3n/2)]]), no próximo 9 vezes n/4 ([[{red}(9n/4)]]) ...

Seguindo esse padrão, temos uma **progressão geométrica**, com a raiz sendo 3/2! Assim:

* [[q = 3/2]]

Tendo essas informações, podemos usar a fórmula de complexidade para PGs:

* ${a1}\cdot{\frac{q^x - 1}{q-1}}$

Onde:
* $a1$ é o primeiro elemento;
* $q$ é a razão;
* $x$ é o número de elementos;

:::

**Passo 3:**

Finalmente já temos todas informações para calcular e simplificar a complexidade do algoritmo. Tente fazer essa parte sozinho, mas lembre-se de verificar o material da disciplina, especialmente as ferramentas matemáticas de manipulação de log! Esse é um passo mais demorado, mas não se preocupe, basta tomar cuidado nas contas e ir com calma ;)!
Algumas das manipulações que faremos são:

* Cancelamento: $b^log_{b}n = n$;
* Inversão de base: $log_{b}a = \frac{1}{log_{a}b}$;

::: Gabarito Passo 3

Agora, basta substituir na equação os valores que já conhecemos:

${n}\cdot{\frac{\frac{3}{2}^{log_{2}n} - 1}{\frac{3}{2}-1}}$

Usando algumas transformações matemáticas e ferramentas que estão no site da disciplina:

{red}(*focando no numerador da fração:*)

$\frac{3}{2}^{log_{2}n} - 1$ --->
$\frac{3^{log_2n}}{2^{log_2n}} - 1$ --->
$\frac{3^{\frac{log_3n}{log_{3}2}}}{n} - 1$ --->
$\frac{n^{\frac{1}{log_{3}2}}}{n} - 1$ --->
$\frac{n^{log_{2}{3}}}{n} - 1$

{red}(*operações matemáticas com toda a equação:*)

${n}\cdot{\frac{\frac{n^{log_{2}{3}}}{n} - 1}{\frac{1}{2}}}$ --->
${n}\cdot{(\frac{n^{log_{2}{3}}}{n} - 1)}\cdot{2}$ --->
${(\frac{n^{log_{2}{3}}}{n}\cdot{n} - {1}\cdot{n})}\cdot{2}$ --->
$2n^{log_{2}{3}} - 2n$

Após todas essa opreações, ficamos com a equação final assim:

[[$2n^{log_{2}{3}} - 2n$]]

Porém, ainda não acabamos. Como vocês sabem, para calcular a complexidade mesmo, precisamos considerar o pior caso possível. Então, apesar de termos o termo $n$, não será ele que vamos considerar. O termo que vamos considerar é o $n^{log_{2}{3}}$.

Portanto, a complexidade do algoritmo é ***$O(n^{log_{2}{3}})$***.
???

??? Checkpoint
Tente agora pensar, qual complexidade é melhor? $O(n)$, que é a complexidade da multiplicação normal, ou $O(n^{log_{2}{3}})$, que é a complexidade do Karatsuba?

::: Gabarito

Agora sabemos a complexidade do algorimto, mas por que ele é melhor que o método normal de multiplicação? Já que $log_23$ é igual a aproximadamente 1,5849, podemos escrever a complexidade do Karatsuba como *$O(n^{1.59})$*. 

A complexidade do método normal é ***$O(n^2)$***, enquanto a complexidade do algoritmo Karatsuba é ***$O(n^{1.59})$***, e portanto, é mais eficiente! Não é uma melhora tão grande, já que existem algoritmos mais recentes e mais eficientes que também realizam multiplicação. Porém, considerando a data que foi desenvolvido e sua complexidade, o algoritmo Karatsuba, sem dúvidas, é um algoritmo que vale a pena estudar para entender um pouco da base de como podemos simplificar operações que apesar de básicas podem complicar muito. 

:::

???

E para finalizar, só queria deixar claro que: as simplificações não param por aí meus colegas. Tem como ir muito além ainda nesse assunto.

![](karatsuba_meme.png)

Desafio/ Sessão extra
-------

!!! Aviso
Se você já terminou as sessões anteriores e ficou curioso para saber como se implementa o código, então aqui está uma sessão extra para entender a lógica do código. Apenas se já terminou, priorize as outras sessões.  
!!!

Nos blocos anteriores você aprendeu e deduziu a fórmula do Karatsuba. Agora, como seria isso em código?


Vimos que o primeiro passo era dividir quem iremos multiplicar em números menores. Sendo assim, precisaremos ir dividindo os números na função até que eles sejam menores que 10, ou seja, possuam um único dígito para multiplicarmos.

Mas antes de fazermos isso, precisaremos determinar quantos dígitos têm os números digitados que serão multiplicados.

??? Checkpoint

Implemente uma função em C, a `md getSize(long num)` que recebe um número do tipo long e que retorna o número de dígitos.  

::: Gabarito
``` c
#include <stdio.h>

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
Se você esqueceu a condição de parada, volte ao início do bloco "*Como o algoritmo funciona?*".
!!!

::: Gabarito
``` c

long karatsuba(long X, long Y)
{
    // Caso base
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

* Por que o maior tamanho dos dígitos entre os dois números?
Para não perder valores e garantir que estaremos fazendo o karatsuba com todos os dígitos de um número. Dessa forma, imagine que vamos multiplicar 13 e 5200000. Se pegássemos o número de dígitos do 13, que é 2, quando fóssemos dividir os números em partes menores (principalmente o 5200000) teríamos problemas.

* Por que dividido por 2?

Pois na fórmula, temos que os dois 10 que aparecem estão elevados a $n/2$. Então isso já ajuda a deixar mais compacto e com menos variáveis na conta final, melhorando a sua vizualização :).


::: Gabarito do gabarito
``` c
#include <math.h>


long karatsuba(long X, long Y)
{
    // Caso base
    if (X < 10 && Y < 10)
        return X * Y;

    // Faz as recursões até o caso base
    int n = (int)(size / 2.0);

    ...

    long ac = karatsuba(a, c);
    long bd = karatsuba(b, d);
    long e = karatsuba(a + b, c + d) - ac - bd;

    // Retorna a equação
    return (long)(pow(10, 2 * n) * ac + pow(10, n) * e + bd);

}

```

:::

???


A única coisa que falta agora em nossa função é determinar os termos que aparecem na equação, e que irão se utilizar do $n$ determinado anteriormente. Caso você tenha esquecido como descrevemos matematicamente cada um dos termos, volte ao bloco anterior.

??? Checkpoint

Por último, implemente em C as definições dos termos $a$, $b$, $c$ e $d$.

!!! Dica
Sempre que você for dividindo os números pela metade, pode fazer uso da potência de 10...
!!!

::: Gabarito

``` c

long karatsuba(long X, long Y)
{
    // Caso base
    if (X < 10 && Y < 10)
        return X * Y;

    // Determine o tamanho de X e Y
    int size = fmax(getSize(X), getSize(Y));

    // Separa o X do Y
    int n = (int)(size / 2.0);
    long p = (long)pow(10, n);
    long a = (long)(X / (double)p);
    long b = X % p;
    long c = (long)(Y / (double)p);
    long d = Y % p;

    // Faz as recursões até o caso base
    long ac = karatsuba(a, c);
    long bd = karatsuba(b, d);
    long e = karatsuba(a + b, c + d) - ac - bd;

    // Retorna a equação
    return (long)(pow(10, 2 * n) * ac + pow(10, n) * e + bd);
}

``` 

:::

???



Fake Quiz
------

Essas serão cenas dos próximos capítulos...

![](hardworking_cat.gif)

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
