--- 
layout: post
title: CentOS start mongrel no boot
tags: 
- CentOS
- Mongrel
- Rails
status: publish
type: post
published: true
meta: 
  _edit_last: "1"
  _syntaxhighlighter_encoded: "1"
  dsq_thread_id: "181630835"
---
Para iniciar, parar ou reiniciar uma aplicação rails no CentOS, basta seguir os seguintes passos.

##Primeiro passo - Criando um script bash

Esse arquivo deve ser salvo ou ter um link simbólico em /etc/init.d/.

###Permissões para no arquivo:

    chmod 754 /etc/init.d/ror

* 7 - Dono do arquivo tem todas permissões sobre o arquivo (leitura, execução e gravação)
* 5 - O grupo poder ler e executar
* 4 - Outros podem ler

###Criar o script

    #!/bin/bash
    #
    # ror
    #
    # chkconfig: 2345 82 82
    # description: ror
    
    function start_rails(){
     echo "Starting click-anywhere-to-leave-a-message.com/current";
     cd "/home/click/rails_apps/click-anywhere-to-leave-a-message.com/current";
     mongrel_rails start -e production -p 12006 -d;
    }
    
    function stop_rails(){
     echo "Stopping click-anywhere-to-leave-a-message.com/current";
     cd "/home/click/rails_apps/click-anywhere-to-leave-a-message.com/current";
     mongrel_rails stop &amp;&amp; rm -rf "/home/click/rails_apps/click-anywhere-to-leave-a-message.com/current/mongrel.pid";
    }
    
    case "$1" in
     start)
     start_rails;
     ;;
    stop)
    stop_rails;
    ;;
    restart)
    stop_rails;
    start_rails;
    ;;
    esac

Para executar.

    mongrel_rails stop && rm -rf "/home/click/rails_apps/click-anywhere-to-leave-a-message.com/current/mongrel.pid";

Tive que colocar o rm -rf mongrel.pid, pois o meu servidor não estava apagando o mongrel.pid em alguns casos.

##Segundo passo - Adicionando o script bash como serviço

    chkconfig --add ror
    chkconfig --list ror
    ror 0:off 1:off 2:on 3:on 4:on 5:on 6:off

##Principais referências

* [Chmod](http://pt.wikipedia.org/wiki/Chmod)
* [CentOS chkconfig](http://www.centos.org/docs/5/html/Deployment_Guide-en-US/s1-services-chkconfig.html)
