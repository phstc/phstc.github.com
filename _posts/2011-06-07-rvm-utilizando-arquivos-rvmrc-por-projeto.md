--- 
layout: post
title: RVM Utilizando arquivos .rvmrc por projeto
tags: 
- .rvmrc
- Ruby
- RVM
---
{% include JB/setup %}

O [rvmrc](https://rvm.beginrescueend.com/workflow/rvmrc#project) por projeto é um recurso muito útil do [Ruby Version Manager (RVM)](https://rvm.beginrescueend.com). Com ele você pode configurar qual [rubie](https://rvm.beginrescueend.com/rubies) e qual [gemset](https://rvm.beginrescueend.com/gemsets/) o seu projeto utilizará, evitando possíveis conflitos entre projetos.
<!--more-->

Para utilizar o rvmrc, basta criar um arquivo .rvmrc na raiz do seu projeto definindo a rubie e gemset serão utilizados.

    rvm gemset create hello_world
    mkdir hello_world
    cd hello_world
    echo 'rvm 1.8.7@hello_world' > .rvmrc

Desta forma o RVM automaticamente utilizará o rubie e gemset definidos no arquivos .rvmc. Sensacional!

Além do rvmrc por projeto, você também pode usar um rvmrc por sistema (default) ou por usuário. Por usuário basta criar o rvmrc na raiz do "$HOME/.rvmrc".

A ordem de precedência do rvmrc é por projeto, por usuário e por último por sistema.

O link abaixo detalha os três tipos de rvmrc.

[rvm.beginrescueend.com/workflow/rvmrc/](https://rvm.beginrescueend.com/workflow/rvmrc)
