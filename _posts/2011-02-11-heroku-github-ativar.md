--- 
layout: post
title: "Heroku + GitHub, ativar"
tags: [Git, GitHub, Heroku]
---
{% include JB/setup %}

Esse post descreve como utilizar o Heroku e GitHub simultaneamente Quem se lembra do [Supergêmeos](http://pt.wikipedia.org/wiki/Superg%C3%AAmeos), ativar?

![Supergêmeos, ativar!](/assets/images/posts/supergemeos-ativar.jpg)

A utilização simultânea do Heroku e GitHub é dada graças ao Git, pois ele permite trabalhar com mais de um server remoto (remote) por repositório. Para isso, após criar um repositório com o Heroku, basta adicionar o GitHub como remote ou vice-versa. 

## Heroku + GitHub, ativar

Quando executamos o comando "heroku create", a gem do Heroku simplesmente adiciona o Heroku como remote.

    $ git remote show heroku
    * remote heroku
      Fetch URL: git@heroku.com:meu-projeto.git
      Push  URL: git@heroku.com:meu-projeto.git
      HEAD branch: master
      Remote branch:
        master tracked
      Local ref configured for 'git push':
        master pushes to master (up to date)

Para adicionar o remote do GitHub, basta executar o [git-remote](http://www.kernel.org/pub/software/scm/git/docs/git-remote.html). 

    # Adicionando o GitHub.
    git remote add github git@github.com:phstc/meu-projeto.git

Na próxima fez que for executar o push basta informar o name do remote desejado.

    # Push no Heroku.
    git push heroku master
    # Push no GitHub.
    git push github master

## Remote origin

Se você adicionar um remote com name "origin", você indicará ao Git que ele é o remote default. Com isso podemos omitir o name quando fizermos um push, pois o Git assumirá o "origin" como default.

    # Adicionando o GitHub como remote default.
    git remote add origin git@github.com:phstc/meu-projeto.git
    # Push no GitHub.
    git push

## Principais referências

* [Documentação oficial](http://www.kernel.org/pub/software/scm/git/docs/)
* [Página na Wikipedia sobre o Git](http://pt.wikipedia.org/wiki/Git)
