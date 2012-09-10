--- 
layout: post
title: "TDC São Paulo 2011: Trilhas Java, TV Digital e Arduino"
tags: [Arduino, Java, TDC 2011, TV Digital]
---
{% include JB/setup %}

Esse ano fui ao [TDC São Paulo 2011](http://www.thedevelopersconference.com.br/tdc/2011/index.html#saopaulo), onde assisti às trilhas de [Java](http://www.thedevelopersconference.com.br/tdc/2011/saopaulo/trilha-java), [TV Digital](http://www.thedevelopersconference.com.br/tdc/2011/saopaulo/trilha-tvdigital) e [Arduino](http://www.thedevelopersconference.com.br/tdc/2011/saopaulo/trilha-arduino). 

Nesse post relato as minhas percepções sobre as trilhas e o evento.

## TDC São Paulo 2011

A organização do evento foi muito boa, atendeu bem as expectativas. O modelo de trilha é bem interessante, você paga somente pelo o que quer assistir. Cada trilha custava 60 reais, porém consegui um código promocional que vi em um tweet que barateou para 49,80 reais. Além da trilha o preço inclui um kit almoço do Subway (lanche, bebida e biscoito).

## Trilha Java

Na trilha Java teve o lançamento mundial do Java 7,  que foi apresentado simultaneamente na Califórnia, Londres e São Paulo. Sendo principalmente apresentado na Califórnia, com chamadas para Londres e São Paulo. Incluir o Brasil no lançamento foi um reconhecimento muito importante para a comunidade Java no Brasil. Sinceramente fiquei orgulhoso. Em Londres a sala era pequena, com poucas pessoas e sem animação. No Brasil era um auditório grande, cheio de pessoas e com muita animação.

A Oracle está fortalecimento o marketing da JVM rodando outras  linguagens. Um dos grandes lançamentos foi o InvokeDynamic, que beneficiará muita outras linguagens sobre a JVM, principalmente em performance e diminuição de complexidade de código. Usaram como exemplo o JRuby, dizendo que poderiam apagar cerca de 2000 classes utilizando esse novo recurso, pois atualmente eles fazem malabarismos para simular métodos dinâmicos.

Acho muito sábia essa campanha da JVM rodando outras   linguagens, pois assim a Oracle poderá continuar forte no mercado, mesmo que não seja com Java.

Java também se beneficiará do InvokeDanymaic, principalmente em substituição ao Reflection. Quase todos os frameworks/projetos atuais em Java utilizam Reflection de alguma forma. O exemplo mais corriqueiro é com Inversão de Controle (IoC).

Outro acontecimento importante que acredito estar totalmente relacionado a  manter-se forte no mercado, foi oficializarem o OpenJDK como Reference Implementation (RI). Com o OpenJDK sendo RI, não existirão mais desculpas do tipo "Não funcionou no OpenJDK", como o caso  clássico do programa do IRPF. Se não funcionar no OpenJDK ou programa está mal feito ou a JVM em que ele foi testado está mal feita.

Java teve algumas atualizações estéticas com o Projeto Coin.  Embora outras alterações bem significativas como lambda, só virão com o Coin 2 no Java 8.

No evento também foi apresentado o NIO.2.

Fiquei contente com as atualizações na linguagem Java, porém acho que o código está ficando esteticamente um Frankenstein.

No InfoQ tem mais detalhes sobre essas atualizações.

* [infoq.com/br/news/2011/07/lancamento-java7](http://www.infoq.com/br/news/2011/07/lancamento-java7)
* [infoq.com/br/articles/java7coin](http://www.infoq.com/br/articles/java7coin)

## Trilha TV Digital

Na trilha de TV Digital assisti apenas à apresentação "Interatividade na TV Digital - Ginga e Plataformas de Desenvolvimento".  Sinceramente, fiquei chateado, pois vi que ainda existe um abismo para desenvolvedores autônomos. NCL, Lua, Ginga etc funcionam para emissoras de TV, porém para fazer um programa em casa e publicar ainda é um processo bem complicado. Acredito que a luz no fim do  túnel só ocorrerá com sistemas operacionais para TV, como o Google TV,  que certamente terá um Apps Marketplace.

## Trilha Arduino

A trilha de Arduino foi OK, exatamente isso, OK. Não foi ruim, porém acredito que tenha faltado um caso real, "Usei para fazer tal protótipo, funcionou bem blah blah...".  Não apenas "dá para fazer isso, dá para fazer aquilo". Comento isso, pois quiseram introduzir o Arduino como algo além de hobby, e infelizmente as palestras só demonstraram hobbies. Claro, a partir desses hobbies podem nascer muitos projetos, porém os apresentados estavam bem "verdes" ainda.

Em resumo, aprendi o que é [XBee](http://en.wikipedia.org/wiki/XBee) e conheci o [Firmata](http://www.arduino.cc/en/Reference/Firmata).
