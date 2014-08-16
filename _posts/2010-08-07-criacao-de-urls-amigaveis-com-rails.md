---
layout: post
title: "Criação de urls amigáveis com Rails"
---

## Atualização do post 18/03/2012

Para implementar urls amigáveis recomendo a gem [FriendlyId](https://github.com/norman/friendly_id). [Ryan Bates](https://twitter.com/#!/rbates) fez um [screencast](http://railscasts.com/episodes/314-pretty-urls-with-friendlyid) bem ditático sobre a gem, recomendo!

--

Para quem usa Rails a partir da versão 2.2 é muito fácil criar urls amigáveis utilizando ActiveSupport::Inflector.parameterize.

    ActiveSupport::Inflector.parameterize "Criação de url amigáveis Rails"
    >> criacao-de-url-amigaveis-rails

O parameterize é um método adicionado diretamente na classe String.

    class Product < ActiveRecord::Base
        def to_param
            "#{id}-#{name.parameterize}"
        end
    end

    Product.create(:name => "Produto Amigável") # Product#1
    product_path(product_1) # /products/1-produto-amigavel
    Product.find("1-produto-amigavel") # Product#1

## ActiveSupport::Inflector

A classe [Inflector](http://api.rubyonrails.org/classes/ActiveSupport/Inflector.html) tem muitos outros métodos que valem a pena serem conferidos.