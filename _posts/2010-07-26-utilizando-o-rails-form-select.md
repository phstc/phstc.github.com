--- 
layout: post
title: "Utilizando o Rails form select"
tags: [form, Rails, select]
---
{% include JB/setup %}

Pesquisei bastante na internet, até achar uma [thread](http://www.ruby-forum.com/topic/160291) demonstrando como utilizar o form select com um array de objetos.

Geralmente meus selects são populados com arrays do [ActiveRecord](http://api.rubyonrails.org/classes/ActiveRecord/Base.html#M001780).

    @users = User.all
    <% form_for @user, :html => {:method => :post} do |f| %>
     <%= f.select :user_id, @users} %>

O select imprimirá com o [to_s](http://ruby-doc.org/core/classes/Object.html#M000359) de cada objeto do array, para exibir o key (id) e value (descrição), basta utilizar o método [collect](http://ruby-doc.org/core/classes/Array.html#M002187), que é uma maneira fácil e rápida para esse propósito.

    <% form_for @user, :html => {:method => :post} do |f| %>
     <%= f.select :user_id, @users.collect {|user| [ user.name, user.id ] } %>

## Outras maneiras para para usar form select

Com um array simples.

    <%= f.select :estados, ['SP', 'PA', 'RJ'] } %>

Com um array com arrays.

    <%= f.select :estados, [['São Paulo', 'SP'], ['Pará', 'PA'], ['Rio de Janeiro', 'RJ']] } %>

## Principais referências

* [Documentação oficial](http://api.rubyonrails.org/classes/ActionView/Helpers/FormOptionsHelper.html)
