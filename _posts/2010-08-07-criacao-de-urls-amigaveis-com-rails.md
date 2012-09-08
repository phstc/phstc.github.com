--- 
layout: post
title: "Cria\xC3\xA7\xC3\xA3o de urls amig\xC3\xA1veis com Rails"
tags: 
- Friendly url
- Slug
- Rails
status: publish
type: post
published: true
meta: 
  _edit_last: "1"
  _syntaxhighlighter_encoded: "1"
  _wp_old_slug: criacao-de-url-amigaveis-com-rails
  dsq_thread_id: "176007105"
---
##Atualização do post 18/03/2012

Para implementar urls amigáveis recomendo a gem [FriendlyId](https://github.com/norman/friendly_id). [Ryan Bates](https://twitter.com/#!/rbates) fez um [screencast](http://railscasts.com/episodes/314-pretty-urls-with-friendlyid) bem ditático sobre a gem, recomendo!

--

Para quem usa Rails a partir da versão 2.2 é muito fácil criar urls amigáveis utilizando ActiveSupport::Inflector.parameterize.

    ActiveSupport::Inflector.parameterize "Criação de url amigáveis Rails"
    >> criacao-de-url-amigaveis-rails

O parameterize é um método adicionado diretamente na classe String.

    class Product < ActiveRecord::Base
        def to_slug
            "#{id}-#{name}".parameterize
        end
    end

##ActiveSupport::Inflector

A classe [Inflector](http://api.rubyonrails.org/classes/ActiveSupport/Inflector.html) tem muitos outros métodos que valem a pena serem conferidos.