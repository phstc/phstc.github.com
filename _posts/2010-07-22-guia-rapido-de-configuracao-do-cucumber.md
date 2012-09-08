--- 
layout: post
title: "Guia r\xC3\xA1pido de configura\xC3\xA7\xC3\xA3o do Cucumber"
tags: 
- BDD
- Cucumber
- Rails
status: publish
type: post
published: true
meta: 
  _edit_last: "1"
  _syntaxhighlighter_encoded: "1"
  _wp_old_slug: cucumber-quicksetup
  dsq_thread_id: "177981018"
---
{% include JB/setup %}

Este post é um guia rápido de configuração do [Cucumber](http://cukes.info/). 

Os passos desses post foram utilizados no projeto [click-anywhere-to-leave-a-message.com](http://click-anywhere-to-leave-a-message.com), disponível para download no [GitHub](http://github.com/phstc/click-anywhere-to-leave-a-message.com).
 
Cucumber é um framework de [Behavior Driven Development](http://en.wikipedia.org/wiki/Behavior_Driven_Development) (BDD), que permite a criação de testes de forma mais legível, usando uma linguagem mais natural chamada [Gherkin](https://github.com/aslakhellesoy/cucumber/wiki/Gherkin).
 
A instalação do Cucumber deve ser seguido pela url oficial do projeto [Cucumber Ruby on Rails](http://wiki.github.com/aslakhellesoy/cucumber/ruby-on-rails).

##Adicionando o Cucumber no projeto Rails

Para começar vamos criar um projeto [Rails](http://rubyonrails.org) e adicionar Cucumber.

    rails click-anyware-to-leave-a-message
    cd click-anyware-to-leave-a-message
    script/generate cucumber

Para testar se o Cucumber foi instalado corretamente.

    rake cucumber

##Descrevendo uma funcionalidade

Após adicionar o Cucumber no projeto Rails ele criará automáticamente o diretório features na raiz do projeto.

A primeira feature que vou criar é **messages.feature** que vai descrever as funcionalidades do [scaffold](http://en.wikipedia.org/wiki/Scaffold_%28programming%29#Scaffolding_in_Ruby_on_Rails) de mensagens do projeto.

Não encontrei nenhum [code convention](http://en.wikipedia.org/wiki/Coding_conventions) para nomes da features, até por que o Cucumber não utiliza [convention over configuration](http://en.wikipedia.org/wiki/Convention_over_configuration) pelo nome da feature para executar o teste, e sim no diretório e extensão do arquivo (meu-projeto/features/\\*.feature).

    Feature: Manage Messages
    In order to create message
    As an author
    I want to create and manage messages
    
    Scenario: List messages
    Given that I have created a message "Olá, essa é a minha primeira mensagem" at position "0", "0"
    When I go to the homepage
    And I should see "Olá, essa é a minha primeira mensagem" at position "0", "0"

###When I go to the homepage

Caso você mude a definição the homepage e o teste pare de funcionar, verifique o arquivo features/support/paths.rb, pois é lá onde estão definidos os paths.

Ao executar pela primera vez os testes retornará a messagem.

    rake cucumber
    1 scenario (1 undefined)
    3 steps (2 skipped, 1 undefined)
    0m0.109s
    
    You can implement step definitions for undefined steps with these snippets:
    
    Given /^that I have created a message "([^\"]*)" at position "([^\"]*)", "([^\"]*)"$/ do |arg1, arg2, arg3|
     pending # express the regexp above with the code you wish you had
    end

Informando que eu tenho que criar o passo.

    Given that I have created a message "Olá, essa é a minha primeira mensagem.

###Criação de passo - step_definitions

Para que o passo funcione, vc pode adicionar diretamente o passo no arquivo features/step_definitions/web_steps.rb ou criar passos por features.

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

##Principais referências

* [Cucumber site oficial](http://cukes.info)
* [rails 2 day 3: behavior-driven development](http://www.ultrasaurus.com/sarahblog/2008/12/rails-2-day-3-behavior-driven-development/#install)
* [Railscasts Beginning with Cucumber](http://railscasts.com/episodes/155-beginning-with-cucumber)

