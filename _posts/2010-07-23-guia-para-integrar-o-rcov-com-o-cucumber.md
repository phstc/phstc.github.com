--- 
layout: post
title: Guia para integrar o RCov com o Cucumber
tags: [Code coverage, Cucumber, RCov, Ruby]
---
{% include JB/setup %}

Esse post é um guia rápido para integrar o [RCov](https://github.com/relevance/rcov) com o [Cucumber](http://cukes.info/). O RCov é uma ferramenta para gerar relatórios de [cobertura de testes](http://en.wikipedia.org/wiki/Code_coverage).

## Instalação do RCov

    gem install rcov

## Integrando o RCov com o Cucumber

Para integrá-los basta modificar a task `cucumber:ok` no arquivo `cucumber.rake` adicionando as instruções abaixo delimitadas por #Inicio RCov e #Fim RCov.

    # cucumber.rake
    Cucumber::Rake::Task.new({:ok => 'db:test:prepare'}, 'Run features that should pass') do |t|
     t.binary = vendored_cucumber_bin # If nil, the gem's binary is used.
     t.fork = true # You may get faster startup if you set this to false
     t.profile = 'default'
     #Início RCov - CODE COVERAGE
     t.rcov = true
     t.rcov_opts = %w{--rails --exclude osx\/objc,gems\/,spec\/,features\/ --aggregate coverage.data}
     t.rcov_opts << %[-o "coverage"]
     #Fim RCov - CODE COVERAGE
    end

Caso você não queira que seja gerada a cobertura de testes todas as vezes que executar os testes, basta criar uma nova task específica para geração da cobertura de testes.

    Cucumber::Rake::Task.new(:rcov) do |t|
     t.rcov = true
     t.rcov_opts = %w{--rails --exclude osx\/objc,gems\/,spec\/,features\/ --aggregate coverage.data}
     t.rcov_opts << %[-o "coverage"]
    end

Para executá-la.

    rake cucumber:rcov

## Sugestão para o .gitignore

Para ignorar os arquivos gerados pelo RCov, basta adicionar as linhas abaixo no arquivo .gitignore.

    coverage/*
    coverage.data

## Principais referências

* [RCov](http://github.com/relevance/rcov)
* [Cucumber](http://cukes.info)