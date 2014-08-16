---
layout: post
title: "CentOS start Mongrel no boot"
---

Este post detalha os passos para iniciar, parar ou reiniciar uma aplicação Rails no CentOS.

## Primeiro passo - Criando um script bash

Esse arquivo deve ser criado ou ter um link simbólico em `/etc/init.d/ror`.

### Permissões para no arquivo:

    chmod 754 /etc/init.d/ror

* 7 - Dono do arquivo tem todas permissões sobre o arquivo (leitura, execução e gravação)
* 5 - O grupo poder ler e executar
* 4 - Outros podem ler

### Script bash

    #!/bin/bash
    #
    # ror
    #
    # chkconfig: 2345 82 82
    # description: ror

    function start_rails(){
     echo "Starting app/current";
     cd "/home/user/rails_apps/app/current";
     mongrel_rails start -e production -p 12006 -d;
    }

    function stop_rails(){
     echo "Stopping app/current";
     cd "/home/user/rails_apps/app/current";
     mongrel_rails stop;
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

## Segundo passo - Adicionando o script bash como serviço

    chkconfig --add ror
    chkconfig --list ror
    ror 0:off 1:off 2:on 3:on 4:on 5:on 6:off

## Principais referências

* [Chmod](http://pt.wikipedia.org/wiki/Chmod)
* [CentOS chkconfig](http://www.centos.org/docs/5/html/Deployment_Guide-en-US/s1-services-chkconfig.html)
