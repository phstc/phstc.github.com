--- 
layout: post
title: "GitHub - Múltiplos forks para o mesmo repositório"
tags: 
- GitHub
---
{% include JB/setup %}

No GitHub não é possível fazer vários forks de um mesmo projeto.

    github.com/user/projetoa
    github.com/user/projetoa(2)
    github.com/user/projetoa(3)

Uma forma de simular isso é com branches.

##Motivação

Fiz um [fork](http://help.github.com/forking/) de um projeto onde fiz muitas alterações, algumas eram interessantes para um [pull request](http://help.github.com/pull-requests/) e outras eram interessantes só para mim. 

Pensei em fazer um segundo fork e subir as alterações para o pull request nesse novo fork. Porém isso não é possível.

##Solução

Para solucionar criei um novo branch "original-nome-do-projeto" no meu fork com HEAD do projeto original.

    $ cd workspace/gesture-translate/gtranslate
    $ git remote add upstream http://github.com/bpierre/gtranslate.git
    $ git fetch upstream
    $ git branch original-gtranslate upstream/master
    $ git checkout original-gtranslate

Fiz as alterações no branch para enviar como pull request.

    $ git commit -a -m 'Mudanças relevantes para o projeto original';
    $ git push origin original-gtranslate

Na página do GitHub mudei do branch master para o branch original-gtranslate.

![](/images/posts/Screen-shot-2010-10-20-at-1.44.03-AM.png)

Fiz um pull request das mudanças feitas no branch original-gtranslate.

![](/images/posts/Screen-shot-2010-10-20-at-1.45.19-AM.png)

Essa solução foi baseada em um [comentário](http://support.github.com/discussions/repos/2420-multiple-forks#comment_958975) do support staff do GitHub.