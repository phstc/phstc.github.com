---
layout: post
title: "Polymorphic associations com Rails"
---


## Polymorphic associations

Se você precisa de um modelo genérico, por exemplo Address, que será associado com User, Customer, Store etc,  polymorphic associations pode ser uma boa opcão.

## Passo 1 - Criar o modelo

    script/generate model Address addressable_type:string addressable_id:integer name:string street1:string street1_number:string street2:string zip_code:string city:string state:string country:string

    rake db:migrate

## Passo 2: Adicionar belongs_to no address.rb

    belongs_to :addressable, :polymorphic => true

## Passo 3: Adicionar o has_many nos models que utilizaram Address

    # customer.rb
    class Customer < ActiveRecord::Base
    has_many :addresses, :as => :addressable, :dependent => :destroy

    # user.rb
    class User < ActiveRecord::Base
    has_many :addresses, :as => :addressable, :dependent => :destroy

    # store.rb
    class Store < ActiveRecord::Base
    has_many :addresses, :as => :addressable, :dependent => :destroy

## Principais referências

* [railsforum.com/viewtopic.php?id=5882](http://railsforum.com/viewtopic.php?id=5882)
* [m.onkey.org/2007/8/14/excuse-me-wtf-is-polymorphs](http://m.onkey.org/2007/8/14/excuse-me-wtf-is-polymorphs)
