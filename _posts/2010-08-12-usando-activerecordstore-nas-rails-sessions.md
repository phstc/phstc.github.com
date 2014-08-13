---
layout: post
title: Usando ActiveRecordStore nas Rails Sessions
---

A sessão padrão do Rails é o Cookie Store, que tem um limite de 4K, sendo usada tipicamente usada para armazenar ids de objetos persistidos na base de dados ou pequenas informações.

Caso você receba um ```ActionController::Session::CookieStore::CookieOverflow``` como o abaixo, você pode tentar o ActiveRecordStore ao invés do Cookie Store.


    /!\ FAILSAFE /!\ Thu Jul 29 21:23:39 -0300 2010
     Status: 500 Internal Server Error
     ActionController::Session::CookieStore::CookieOverflow
    /System/Library/Frameworks/Ruby.framework/Versions/1.8/usr/lib/ruby/gems/1.8/gems/actionpack-2.3.5/lib/action_controller/session/cookie_store.rb:102:in `call'

(eu só consegui resolver esse problema removendo os cookies do browser, reiniciar o servidor não resolveu para mim.)

##ActiveRecordStore

O ActiveRecordStore é mais lento que o Cookie Store, porém seu tamanho só é limitado pela base dados.

Para usar o ActiveRecordStore basta adicionar a seguinte linha no environment.rb.

     config.action_controller.session_store = :active_record_store

Criar as tabelas para armazenar a sessão.

    rake db:sessions:create

Esse é um exemplo básico para usar o ActiveRecordStore, na documentação oficial [HowtoChangeSessionStore](http://oldwiki.rubyonrails.org/rails/pages/HowtoChangeSessionStore) tem outras opções de configuração, assim como outras opções de persistência para a sessão.
