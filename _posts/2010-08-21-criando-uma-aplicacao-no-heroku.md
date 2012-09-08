--- 
layout: post
title: "Criando uma aplica\xC3\xA7\xC3\xA3o no Heroku"
tags: 
- Heroku
- Rails
- Ruby
status: publish
type: post
published: true
meta: 
  _syntaxhighlighter_encoded: "1"
  _edit_last: "1"
  _wp_old_slug: ""
  dsq_thread_id: "176942492"
---
##Breve introdução

[Heroku](http://heroku.com/) é um cloud [PaaS](http://en.wikipedia.org/wiki/Platform_as_a_service), muito fácil para utilizar em projetos Ruby e agora com o novo stack o [Cedar](https://devcenter.heroku.com/articles/cedar), também é possível utilizá-lo com outras linguagens e plataformas.

A idéia de um PaaS é de abstrair a complexidade de um sysadmin, toda a parte de hardware e configuração do servidor não fica sobe a responsabilidade de quem desenvolve, fica por conta de quem provê o serviço.

##O que precisa para funcionar?

* Ruby on Rails instalado
* Git instalado
* [Ler o Getting Started with Heroku](https://devcenter.heroku.com/articles/quickstart)
* Ter [uma conta no Heroku](http://heroku.com/signup)
* Ter o [Heroku toolbelt](https://toolbelt.heroku.com/) ou a [gem Heroku](http://rubygems.org/gems/heroku)
* Configurar as chaves ssh: [Mac](http://help.github.com/mac-key-setup/), [Windows](http://help.github.com/msysgit-key-setup/), [Linux](http://help.github.com/linux-key-setup/)

##Criando uma aplicação básica

    rails myapp
    cd myapp
    git init
    git add .
    git commit -a -m 'first commit'
    heroku create myapp
    git push heroku master

##Configurando gems

Se a sua aplicação usar gems, basta configurar um arquivo [.gems](http://docs.heroku.com/gems) ou Gemfile que o Heroku se encarregará de instalar as gems.

    touch .gems
    git add .gems
    git commit -a -m "Adding .gems"
    git push heroku master
    cat .gems
    rails -v 2.3.5
    geokit -v 1.5.0

##Para executar os migrates

    heroku rake db:migrate
