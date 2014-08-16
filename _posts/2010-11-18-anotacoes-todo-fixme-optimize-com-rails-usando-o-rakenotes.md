---
layout: post
title: "Anotações TODO, FIXME, OPTIMIZE com Rails usando o rake:notes"
---

Comentários do tipo TODO são bastante úteis para deixar pontos de atenção no seu código.

    # TODO Inserir validação caso o input seja nulo.
    def some_method(input)
    end

Algumas IDEs como Eclipse e Netbeans possuem uma view "Tasks", que exibe estes comentários.

Com Rails a visualização dessas comentários pode ser feita por linha de comando usando `rake:notes`.

## TODO, FIXME, OPTIMIZE

Rails além de comentário do tipo TODO, possui o FIXME e OPTIMIZE.

Para visualizar os comentários:

    # Exibe listagem com todos os comentários TODO, FIXME e OPTIMIZE.
    rake notes
    # Filtra por comentários TODO.
    rake notes:todo
    # Filtra por comentários FIXME.
    rake notes:fixme
    # Filtra por comentários OPTIMIZE.
    rake notes:optimize

## Exemplo real

    # app/models/article.rb
    class Article < ActiveRecord::Base

     # TODO add named_scopes
     def some_method1
       # ...
     end

     # FIXME method A is broken
     def method_a
       # ...
     end

     # OPTIMIZE improve the code
     def some_method2
       # ...
     end
    end

    $ rake notes
    app/models/article.rb:
      * [4] [TODO] add named_scopes
      * [9] [FIXME] method A is broken
      * [14] [OPTIMIZE] improve the code

Os valores 4, 9 e 14 são as linhas onde estão os respectivos comentários.

## CUSTOM

Existe uma opção muito interessante que é o `rake notes:custom`, que permite filtrar por comentários específicos.

    # Filtra por comentários COLL
    rake notes:custom ANNOTATION=COOL

## Qual usar?

Eu particularmente acho confuso ter várias tipos de anotações. Acho que TODO é bem genérico e atende muito bem como lembrete. Lembretes que devem ser vistos em algum momento, como nas folgas dos projetos e não serem usados apenas para aliviar a consciência.

## Principais referências

* [rubyquicktips.tumblr.com/post/385665023/fixme-todo-and-optimize-code-comments](http://rubyquicktips.tumblr.com/post/385665023/fixme-todo-and-optimize-code-comments)
* [blog.rubyyot.com/2009/05/gtd-on-rails-with-annotations/](http://blog.rubyyot.com/2009/05/gtd-on-rails-with-annotations)
* [ryandaigle.com/archives/2007/2](http://ryandaigle.com/archives/2007/2)
* [rbjl.net/13-organise-your-code-comments-with-rake-notes-todo](http://rbjl.net/13-organise-your-code-comments-with-rake-notes-todo)
