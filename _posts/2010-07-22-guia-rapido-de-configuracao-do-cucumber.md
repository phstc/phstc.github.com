--- 
layout: post
title: "Guia rápido de configuração do Cucumber"
tags: [BDD, Cucumber, Rails]
---
{% include JB/setup %}

<del>Os passos desses post foram utilizados no projeto [click-anywhere-to-leave-a-message.com](http://click-anywhere-to-leave-a-message.com), disponível para download no [GitHub](http://github.com/phstc/click-anywhere-to-leave-a-message.com).</del>
 
[Cucumber](http://cukes.info/) é um framework de [Behavior Driven Development](http://en.wikipedia.org/wiki/Behavior_Driven_Development) (BDD), que permite a criação de testes automatizados por comportamento usando uma linguagem mais natural chamada [Gherkin](https://github.com/aslakhellesoy/cucumber/wiki/Gherkin).

> Cucumber lets software development teams describe how software should behave in plain text. The text is written in a business-readable domain-specific language and serves as documentation, automated tests and development-aid - all rolled into one format.
 
A instalação do Cucumber deve ser seguida pela url oficial do projeto [Cucumber Ruby on Rails](http://wiki.github.com/aslakhellesoy/cucumber/ruby-on-rails).

## Adicionando o Cucumber em um projeto Rails

Para começar vamos criar um projeto [Rails](http://rubyonrails.org) e adicionar Cucumber.

    rails click-anyware-to-leave-a-message
    cd click-anyware-to-leave-a-message
    script/generate cucumber

Para testar se o Cucumber foi instalado corretamente.

    rake cucumber

## Descrevendo uma funcionalidade

Após adicionar o Cucumber será criado o diretório features na raiz do projeto. O Cucumber busca pelo pattern `meu-projeto/features/\\*.feature` para executar as features.

A primeira feature será a `messages.feature`, que descreverá as funcionalidades do ["scaffold"](http://en.wikipedia.org/wiki/Scaffold_%28programming%29#Scaffolding_in_Ruby_on_Rails) de mensagens do projeto.

    Feature: Manage Messages
    In order to create a new message
    As an author
    I want to create and manage messages
    
    Scenario: List messages
    Given that I have created a message "Olá, essa é a minha primeira mensagem" at position "0", "0"
    When I go to the homepage
    And I should see "Olá, essa é a minha primeira mensagem" at position "0", "0"

### When I go to the homepage

Caso você mude o path `the homepage` e o teste pare de funcionar, verifique o arquivo `features/support/paths.rb`, pois é lá onde estão definidos os paths.

Ao executar pela primeira vez os testes o Cucumber retornará:

    rake cucumber
    1 scenario (1 undefined)
    3 steps (2 skipped, 1 undefined)
    0m0.109s
    
    You can implement step definitions for undefined steps with these snippets:
    
    Given /^that I have created a message "([^\"]*)" at position "([^\"]*)", "([^\"]*)"$/ do |arg1, arg2, arg3|
     pending # express the regexp above with the code you wish you had
    end

Informando que eu tenho que criar o passo:

    Given that I have created a message "Olá, essa é a minha primeira mensagem.

### Criação do passo

Para que o passo funcione, você pode adicioná-lo diretamente no arquivo `features/step_definitions/web_steps.rb` ou criar um novo arquivo de step definitions específico.

    # features/step_definitions/messages_steps.rb
    Given /^that I have created a message "([^\"]*)" at position "([^\"]*)", "([^\"]*)"$/ do |msg, top, left|
     Message.create!(
     :message => msg,
     :author => 'session_id',
     :ip => '127.0.0.1',
     :left => left,
     :top => top,
     :status => Message::SAVED_BY_THE_USER)
    end

## Principais referências

* [Cucumber site oficial](http://cukes.info)
* [rails 2 day 3: behavior-driven development](http://www.ultrasaurus.com/sarahblog/2008/12/rails-2-day-3-behavior-driven-development/#install)
* [Railscasts Beginning with Cucumber](http://railscasts.com/episodes/155-beginning-with-cucumber)

