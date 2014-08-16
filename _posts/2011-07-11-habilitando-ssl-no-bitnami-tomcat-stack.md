---
layout: post
title: "Habilitando SSL no BitNami Tomcat Stack"
---

Em continuação ao [Primeiros passos no Amazon EC2](http://pablocantero.com/blog/2011/07/08/primeiros-passos-no-amazon-ec2/), este post descreve os passos para habilitar SSL no BitNami Tomcat Stack.

<!--more-->

##Habilitando SSL

Não vou detalhar muito os passos abaixo, pois esses detalhes podem ser encontrados facilmente na internet. Este é um guia copy&paste para criação de SSL self-signed usando o OpenSSL no BitNami Tomcat Stack. Na verdade esses passos funcionam para Ubuntu e Apache2, não necessariamente só para o Tomcat Stack.

##Criando um certificado self-signed

    # Substituindo o usuário corrente (bitnami) para o root.
    $ sudo su root

    # Instalando o OpenSSL.
    $ apt-get install openssl

    # Criando o diretório para guardar os certificados.
    $ cd ~
    $ mkdir sslcert
    $ cd sslcert

    # Criação das chaves pública e privada.
    $ openssl genrsa -des3 -out server.key 1024

    # No "Common Name (eg, YOUR name)" preencher com o domínio ou ip que o servidor que será acessado.
    $ openssl req -new -key server.key -out server.csr

    # O parâmetro "-days 1825", quer dizer que a chave só expirará em 5 anos.
    $ openssl x509 -req -days 1825 -in server.csr -signkey server.key -out server.crt

    # Criando um chave sem senha, caso contrário o Apache sempre pedirá ao iniciar a senha definida na criação do server.key.
    $ cp server.key server.key.secure
    $ openssl rsa -in server.key.secure -out server.key

    # Colocando permissões somente de leitura para o owner do arquivo e sem permissão para os demais.
    $ chmod 400 server.csr
    $ chmod 400 server.crt
    $ chmod 400 server.key
    $ chmod 400 server.key.secure

    # Copiando arquivos necessários para o diretório do Apache.
    $ mkdir /opt/bitnami/apache2/conf/sslcert
    $ cp server.crt  /opt/bitnami/apache2/conf/sslcert
    $ cp server.key  /opt/bitnami/apache2/conf/sslcert

##Habilitando SSL no Apache

Descomentar a linha ```Include conf/extra/httpd-ssl.conf``` no ```/opt/bitnami/apache2/conf/httpd.conf```.

Alterar as linhas no arquivo ```/opt/bitnami/apache2/conf/extras/httpd-ssl.conf```.

De

    SSLCertificateFile "/opt/bitnami/apache2/conf/server.crt"

Para

    SSLCertificateFile "/opt/bitnami/apache2/conf/sslcert/server.crt"

De

    SSLCertificateKeyFile "/opt/bitnami/apache2/conf/server.key"

Para

    SSLCertificateKeyFile "/opt/bitnami/apache2/conf/sslcert/server.key"

Basta reiniciar o Apache que o SSL estará pronto para o uso.

    apachectl restart
