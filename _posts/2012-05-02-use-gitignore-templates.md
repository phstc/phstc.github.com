---
layout: post
title: Use .gitignore templates!
---

Com o tempo acabamos criando alguns arquivos [.gitignore](http://help.github.com/ignore-files) de acordo com o environment que trabalhamos. Geralmente quando eu inicio um novo projeto eu copio o .gitignore de um projeto similar que eu já trabalhei.

    $ cp ~/workspace/old_rails_project/.gitignore ~/workspace/new_rails_project

Funciona bem, mas recentemente encontrei o projeto [.gitignore templates](https://github.com/github/gitignore) que ajuda nesse processo, podendo até evitar que arquivos indesejados passem despercebidos nos nossos arquivos .gitignore.

##.gitignore templates

O projeto contém vários templates de arquivos .gitignore por tipo de projeto/framework e globais por sistema operacional e editores.

[Java.gitignore](https://github.com/github/gitignore/blob/master/Java.gitignore)

    *.class

    # Package Files #
    *.jar
    *.war
    *.ear

Os arquivos .gitignore globais ficam separados na pasta [Global](https://github.com/github/gitignore/tree/master/Global) do projeto.

[OSX.gitignore](https://github.com/github/gitignore/blob/master/Global/OSX.gitignore)

    .DS_Store

    # Thumbnails
    ._*

    # Files that might appear on external disk
    .Spotlight-V100
    .Trashes

###.gitignore global

Uma boa prática se você quiser ignorar arquivos do seu environment, isso quer dizer do seu sistema operacional, editor etc, que é específico para você e não para o projeto é colocá-los no .gitignore global do seu Git.

    $ touch ~/.gitignore_global
    Adicione suas configurações, arquivos temporários do vim, do OSX etc.
    $ git config --global core.excludesfile ~/.gitignore_global

##gemignore

Para facilitar ainda mais a vida, criaram a gem [gemignore](https://github.com/x3ro/gemignore), que adiciona snippets do .gitignore templates no arquivo .gitignore do diretório atual.

Instalando a gem.

    $ gem install gemignore

Adicionando um template.

    $ gemignore add Rails

Adicionado um template Global.

    $ gemignore add Global/OSX

O comando ```add``` dá um append no arquivo ```.gitignore``` do diretório atual. Caso não exista exibirá uma mensagem informando que não encontrou o arquivo.

