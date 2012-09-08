--- 
layout: post
title: Utilizando o form select Rails
tags: 
- form
- Rails
- select
status: publish
type: post
published: true
meta: 
  _wp_old_slug: ""
  _syntaxhighlighter_encoded: "1"
  _edit_last: "1"
  dsq_thread_id: "175176695"
---
Pesquisei bastante na internet, até achar uma [thread](http://www.ruby-forum.com/topic/160291) desmontrando como utilizar o form select com um array de objetos com key e value.

Geralmente meus selects são populados com arrays de [ActiveRecord](http://api.rubyonrails.org/classes/ActiveRecord/Base.html#M001780).

    @users = User.all

Se eu fizer diretamente.

    <% form_for @user, :html => {:method => :post} do |f| %>
     <%= f.select :user_id, @users} %>

O select imprimirá com o [to_s](http://ruby-doc.org/core/classes/Object.html#M000359) de cada objeto do array, para filtrar com key (id) e value (descrição), basta utilizar o método [collect](http://ruby-doc.org/core/classes/Array.html#M002187), que é uma maneira fácil e rápida para esse propósito.

    <% form_for @user, :html => {:method => :post} do |f| %>
     <%= f.select :user_id, @users.collect {|user| [ user.name, user.id ] } %>

##Outras maneiras para para usar form select

Com um array.

    <%= f.select :estados, ['SP', 'PA', 'RJ'] } %>

Com um array de array para termos key e value.

    <%= f.select :estados, [['São Paulo', 'SP'], ['Pará', 'PA'], ['Rio de Janeiro', 'RJ']] } %>

##Principais referências

* [Documentação oficial](http://api.rubyonrails.org/classes/ActionView/Helpers/FormOptionsHelper.html)
