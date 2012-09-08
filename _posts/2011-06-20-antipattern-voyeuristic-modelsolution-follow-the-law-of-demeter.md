--- 
layout: post
title: "AntiPattern: Voyeuristic Model/Solution: Follow the Law of Demeter"
tags: 
- AntiPattern
- Delegate
- Rails
status: publish
type: post
published: true
meta: 
  _edit_last: "1"
  dsq_thread_id: "337523755"
  _syntaxhighlighter_encoded: "1"
---
Esse post é um pedacinho do capitulo 1, do livro [Rails Antipatterns](http://www.amazon.com/Rails-AntiPatterns-Refactoring-Addison-Wesley-Professional/dp/0321604814). Focando no uso do método [delegate](http://api.rubyonrails.org/classes/Module.html#method-i-delegate) do Rails, referênciado no tópico Solution: Follow the [Law of Demeter](http://en.wikipedia.org/wiki/Law_of_Demeter).

<!--more-->
Antes de iniciar a explicação sobre o delegate, vamos a definição de [Antipattern](http://en.wikipedia.org/wiki/Anti-pattern).

> In software engineering, an anti-pattern (or antipattern) is a pattern  that may be commonly used but is ineffective and/or counterproductive in  practice. The term was coined in 1995 by Andrew Koenig, inspired by Gang of  Four's book Design Patterns, which developed the concept of design  patterns in the software field. The term was widely popularized three  years later by the book AntiPatterns, which extended the use of the term  beyond the field of software design and into general social  interaction. According to the authors of the latter, there must be at  least two key elements present to formally distinguish an actual  anti-pattern from a simple bad habit, bad practice, or bad idea:
> 
> Some repeated pattern of action, process or structure that initially  appears to be beneficial, but ultimately produces more bad consequences  than beneficial results, and
> 
> A refactored solution exists that is clearly documented, proven in actual practice and repeatable.

Definição de [Law of Demeter](http://en.wikipedia.org/wiki/Law_of_Demeter).

> The Law of Demeter (LoD) or Principle of Least Knowledge is a design guideline for developing software, particularly object-oriented programs. In its general form, the LoD is a specific case of loose coupling. The guideline was invented at Northeastern University towards the end of 1987, and can be succinctly summarized in one of the following ways: Each unit should have only limited knowledge about other units: only units "closely" related to the current unit.
> 
> Each unit should only talk to its friends; don't talk to strangers.
> 
> Only talk to your immediate friends.

##Follow the Law of Demeter

Uma Invoice tem um Customer que tem um Address. Este é um exemplo clássico de associações em desenvolvimento de software.

##Implementação trivial em Rails

    class Address < ActiveRecord::Base
      belongs_to :customer
    end
    
    class Customer < ActiveRecord::Base
      has_one :address
      has_many :invoices
    end
    
    class Invoice < ActiveRecord::Base
      belongs_to :customer
    end

Exemplo de view acessando esses modelos.

    <%= @invoice.customer.name %>
    <%= @invoice.customer.address.street %>

Em boa parte dos sistemas, essa implementação funcionará perfeitamente. Porém com um pequeno refactoring, aumentaremos muito a capacidade de evolução desse código (extensibility e maintainability).

Se um dia o relacionamento de Invoice com Customer ou de Customer com Address for alterado, poderá implicar na alteração em todas as referências para Customer ou Address a partir de Invoice.

##Implementação trivial de encapsulamento

    class Invoice < ActiveRecord::Base
      belongs_to :customer
    
      def customer_name
        customer.name
      end
    
      def customer_street
        customer.street
      end
    end
    
    class Customer < ActiveRecord::Base
      belongs_to :address
    
      def street
        address.street
      end
    end

Exemplo de view acessando esses modelos.

    <%= @invoice.customer_name %>
    <%= @invoice.customer_street %>

O exemplo acima resolverá o problema encapsulando as referências de Customer e Address, porém gerará muito código, pois teremos que reescrever todos os getters e setters. Para contornar isso, podemos usar o método delegate do Rails.

##Implementação utilizado o método delegate

    class Invoice < ActiveRecord::Base
      belongs_to :customer
      delegate :name,
               :street,
               :to     => :customer,
               :prefix => true
    end
    
    class Customer < ActiveRecord::Base
      belongs_to :address
      delegate :street,
               :to     => address,
               :prefix => false
    end

Exemplo de view acessando esses modelos.

    <%= @invoice.customer_name %>
    <%= @invoice.customer_street %>

Utilizando o método delegate os modelos ficaram muito mais enxutos e com grande capacidade evolutiva.
