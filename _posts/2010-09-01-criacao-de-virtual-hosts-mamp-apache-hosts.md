---
layout: post
title: Criação de Virtual Hosts: MAMP + Apache + Hosts
---

O acrônimo [MAMP](http://www.mamp.info) é um programa free para o Mac OSX que instala e configura automáticamente o Apache, MySQL, PHP e phpMyAdmin.

* __M__ac OSX
* __A__pache
* __M__ySQL
* __P__HP

## Virtual Hosts

Primeiro passo é configurar o arquivos hosts para resolver nomes como projeto1.local, projeto2.local etc como 127.0.0.1.

    $ sudo vi /private/etc/hosts##
    # Host Database
    #
    # localhost is used to configure the loopback interface
    # when the system is booting.  Do not change this entry.
    ##
    127.0.0.1 localhost
    255.255.255.255 broadcasthost
    ::1 localhost
    fe80::1%lo0 localhost
    127.0.0.1 projeto1.local
    127.0.0.1 projeto2.local
    # ...

Lembrando que para salvar e sair do vi o comando é ESC + ":" + wq!.

Feito isso o MAC irá resolver [projeto1.local](http://projeto1.local) como [127.0.0.1](http://127.0.0.1).

Para adicionar o Virtual Host basta adicionar no ```/Applications/MAMP/conf/apache/httpd.conf``` as entradas abaixo.

    NameVirtualHost *

    <VirtualHost *>
     DocumentRoot "/Applications/MAMP/htdocs"
     ServerName localhost
    </VirtualHost>

    <VirtualHost *>
     DocumentRoot "/Users/pablo/workspace/projeto1"
     ServerName projeto1
    </VirtualHost>

    <VirtualHost *>
     DocumentRoot "/Users/pablo/workspace/projeto2"
     ServerName projeto2
    </VirtualHost>

Pronto! Agora basta acessar [projeto1.local](http://projeto1.local).

## Principais referências

* [sawmac.com/mamp/virtualhosts/](http://www.sawmac.com/mamp/virtualhosts/index.php)

* [support.apple.com/kb/TA27291?viewlocale=en_US](http://support.apple.com/kb/TA27291?viewlocale=en_US)
