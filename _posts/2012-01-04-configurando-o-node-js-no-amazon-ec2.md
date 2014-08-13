---
layout: post
title: Configurando o Node.js no Amazon EC2
---

Esse post é <del>copiado</del> baseado no post [DIY node.js server on Amazon EC2](http://cuppster.com/2011/05/12/diy-node-js-server-on-amazon-ec2/), que foi a melhor referência que encontrei de como configurar um servidor [node.js](http://nodejs.org) completo em uma instância do EC2, no caso uma instância com Ubuntu (ubuntu-maverick-10.10-i386-server).

<!--more-->

Como eu disse estou me baseando com apenas pequenas adaptações no post DIY node.js server on Amazon EC2. Este post me motivou, pois ele funciona, é quase um script, vc pode copiar e colar os trechos de código que funciona. :)

## Seleção de instância no EC2

Eu criei uma instância ubuntu-maverick-10.10-i386-server no EC2, porém deverá funcionar com outras versões do Ubuntu e até em outros Linux com pequenas adaptações.

## Configuração do ambiente

    $ sudo apt-get update
    $ sudo apt-get -y upgrade
    $ sudo apt-get install rcconf
    $ sudo apt-get install build-essential
    $ sudo apt-get install libssl-dev
    $ sudo apt-get install git-core

## Instalação do node.js


O make demora bastante. ;)

    $ wget http://nodejs.org/dist/node-latest.tar.gz
    $ tar xzf node-latest.tar.gz
    $ cd node-v0.6.2
    $ ./configure --prefix=/usr
    $ make
    $ sudo make install

Quando eu escrevi esse post o node-latest era o 0.6.2, talvez a última versão já seja outra, verifique na hora de executar o "cd node-v0.6.2". Vc também pode instalar baixando pelo git, assim vc poderá escolher a versão que quer instalar.

## Instalação do node.js pelo git

    $ git clone git://github.com/ry/node.git
    $ # git checkout origin/v0.4  #OPCIONAL
    $ cd node
    $ ./configure --prefix=/usr
    $ make
    $ sudo make install

## Instalação no npm

    $ cd ~
    $ git clone http://github.com/isaacs/npm.git
    $ cd npm
    $ sudo make install

## Instalação do nginx

    sudo apt-get install nginx

## Configuração do nginx

    sudo vi /etc/nginx/sites-enabled/default
    location / {
            proxy_pass      http://127.0.0.1:3000/;
    }

    location ^(/images|/javascripts|/press|/stylesheets|/templates)$ {
            root  /home/ubuntu/www/public;
            autoindex on;
    }

Para o primeiro location verifique se a porta é mesma que você subirá o seu servidor node.js, a porta default é 3000.

O segundo location é para fazer com que o nginix responda pelo o conteúdo estático sem passar pelo servidor node.js.

## Integrando com o git

Uma coisa bacana é integrar o seu servidor como o git, transformando-o em um remote do git. Fazendo como o Heroku faz, que ao receber um git push, faz o deploy da aplicação.

Para isso:

    $ mkdir ~/www
    $ mkdir ~/repo
    $ cd ~/repo
    $ git init --bare
    $ cat > hooks/post-receive

    #!/bin/sh
    GIT_WORK_TREE=/home/ubuntu/www
    export GIT_WORK_TREE
    git checkout -f

    $ chmod +x hooks/post-receive

## Adicionando sua chave pública no servidor

No sua MAQUINA, LOCALHOST.

Troque o amazon-instance-public-dns pelo o public-dns da sua instância.

    $ cat ~/.ssh/id_rsa.pub | ssh -i amazon-generated-key.pem ec2-user@amazon-instance-public-dns "cat>> ~/.ssh/authorized_keys"

    $ mkdir helloworld
    $ cd helloworld
    $ git init
    $ git remote add ec2 ssh://ubuntu@your.static.ip/home/ubuntu/repo

    $ cat > server.js

    var http = require('http');
    http.createServer(function (req, res) {
      res.writeHead(200, {'Content-Type': 'text/plain'});
      res.end('Hello World\n');
    }).listen(3000, "127.0.0.1");
    console.log('Server running at http://127.0.0.1:3000/');

    $ git add server.js
    $ git commit -m 'first'
    $ git push ec2 +master:refs/heads/master

Pronto, se tudo funcionou como deveria, você conseguirá acessar pelo navegador http://amazon-instance-public-dns.

## Mantendo o seu node.js sempre de pé

Para manter o node.js sempre de pé em caso de reboot ou crashing, estou usando o supervisor, exatamente igual o post que estou <del>copiando</del> me baseando.

    $ sudo apt-get install python-setuptools
    $ sudo easy_install supervisor
    $ curl https://raw.github.com/gist/176149/88d0d68c4af22a7474ad1d011659ea2d27e35b8d/supervisord.sh > supervisord
    $ chmod +x supervisord
    $ sudo mv supervisord /etc/init.d/supervisord

## Marque o supervisor como serviço

    sudo rcconf

## Configurando o supervisor

    $ sudo echo_supervisord_conf > supervisord.conf
    $ sudo mv supervisord.conf /etc/supervisord.conf
    $ sudo vi /etc/supervisord.conf

Ainda no arquivo supervisord.conf, na seção [supervisord] configure:

    [supervisord]
    ;...
    user=ubuntu

Logo acima da seção [program:theprogramname] adicione:

    [program:node]
    command=node app.js
    directory=/home/ubuntu/www
    environment=NODE_ENV=production

    ;[program:theprogramname]

Reload no supervisor.

    $ /etc/init.d/supervisord restart

ou

    $ supervisorctl reload

## Integrando o supervisor com o git

    $ vi ~/repo/hooks/post-receive

    #!/bin/sh
    GIT_WORK_TREE=/home/ubuntu/www
    export GIT_WORK_TREE
    git checkout -f
    sudo supervisorctl restart node

Seguindo os passos acima o seu servidor node.js deverá funcionar direitinho no EC2. Boa sorte!

