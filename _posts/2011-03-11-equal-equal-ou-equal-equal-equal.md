--- 
layout: post
title: Equal, equal ou equal, equal, equal
tags: 
- JavaScript
---
{% include JB/setup %}

##É igual, mas é diferente

[JavaScript](http://pt.wikipedia.org/wiki/JavaScript) permite comparar a igualdade de valores usando ==, ===, !==, !===. Esses operadores podem ter um comportamento igual em muitos casos, porém possuem uma sutil diferença que pode gerar muita dor de cabeça.
<!--more-->

##Equal, equal

Os operadores == e != fazem a comparação **sem checagem de tipo**.

    if(1 == 1){
      // Vai passar por aqui
    }
    if('1' == 1){
      // Vai passar por aqui
    }

##Equal, equal, equal

Os operadores === e !=== fazem a comparação **com a checagem de tipo**.

    if(1 === 1){
      // Vai passar por aqui
    }
    if('1' === 1){
      // Não vai passar por aqui
    }

##Sutil diferença

Nos exemplos acima provavelmente não teremos nenhum problema, porém temos que levar em consideração algumas peculiaridades do JavaScript.

No JavaScript uma string vazia '' e o número 0 são considerados FALSE.

    if(''){
      // Não vai passar por aqui
    }
    if(0){
      // Não vai passar por aqui
    }

Portanto se fizermos as seguintes comparações.

    if('' == false){
      // Vai passar por aqui
    }
    if('0' == false){
      // Vai vai passar por aqui
    }
    if(null == undefined){
      // Vai vai passar por aqui
    }

Lembrando que null (sem valor) e undefined (não declarado ou não instanciado) não passam em ifs, mas não são considerados FALSE tanto usando == quanto ===.

    if(null){
      // Não vai passar aqui
    }
    if(undefined){
      // Não vai passar aqui
    }
    // Mesmo assim não &quot;iguais&quot; a false
    if(null == false || null === false){
      // Não vai passar aqui
    }
    if(undefined == false || undefined === false){
      // Não vai passar aqui
    }

Para evitarmos esse tipo de comportamento podemos fazer a comparação com checagem de tipo.

    if('' === false){
      // Não vai passar por aqui
    }
    if('0' === false){
      // Não vai vai passar por aqui
    }
    if(null === undefined){
      // Não vai vai passar por aqui
    }

##Conclusão

Segundo Douglas Crockford's o autor do livro [O melhor do JavaScript](http://www.submarino.com.br/produto/1/21471189/melhor+do+javascript,+o).

>> JavaScript has two sets of equality operators: === and !==, and their evil twins == and !=.  The good ones work the way you would expect.  If the two operands are of the same type and have the same value, then === produces true and !== produces false.    The evil twins do the right thing when the operands are of the same   type, but if they are of different types, they attempt to coerce the   values.  the rules by which they do that are complicated and unmemorable

Eu particularmente uso equal equal, pois tive problemas com algum navegador, acredito que alguma versão do IE não reconhece equal equal equal.