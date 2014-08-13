---
layout: post
title: Primeiros passos no Amazon EC2
---

## Por que estou migrando para a Amazon?

Há alguns anos utilizo uma maquina virtual (VM) com o CentOS hospedada fora do Brasil, obviamente está fora devido ao custo favorável. Nessa VM estou rodando esse blog, uma aplicação Rails, uma aplicação Java e algumas vezes utilizo para testes diversos. Apesar do ambiente misto com PHP, Ruby e Java, essas aplicações não consomem muito processamento e nem memória.

<!--more-->

Além do preço favorável, outro fator que me motivou para iniciar essa migração para a Amazon, foi a facilidade para gerenciar as maquinas virtuais e recursos. Posso subir uma instância para o blog, outra para Rails, outra para Java, facilmente pelo Web Console da Amazon ou por linha de comando ou por API.

Se tudo estivesse bem na minha VM, eu não teria iniciado essa migração. Porém fizeram uma alteração na infraestrutura de onde hospedo minha VM, que deixou o meu Tomcat muito instável. Para resolver essa instabilidade eu teria que aumentar a memória da VM, implicando no aumento da mensalidade, para mim não faz sentido, pois antes funciona "perfeitamente". Enfim, vamos lá... também não deixa de ser uma oportunidade para aprender um pouco sobre o [Amazon EC2](http://aws.amazon.com/ec2/).

Esse post é focado no primeiros passos que utilizei no EC2 para migrar minha aplicação Java, que apesar de ser super simples, pouco mais que um Web Service, é uma aplicação bastante importante para o meu cliente. Ela foi o ponta pé inicial para essa migração, pois meu cliente estava sendo bastante afetado com a instabilidade do Tomcat.

## Quanto custa?

O EC2 pode ser usado gratuitamente por durante 1 ano ([AWS Free Usage Tier](http://aws.amazon.com/free/)). A versão gratuita já é bastante interessante. O poder de processamento dessa versão supera o poder de processamento da minha VM com CentOS.

Após o período gratuito ou caso você opte por algum plano pago, os valores são bem atrativos. Com planos a partir de 0.02 dólares por hora. Com o plano mais barato, caso a instância seja utilizada 24/7, o custo máximo será por volta de 14 dólares mensais. Para maiores detalhes [Amazon EC2 Pricing](http://aws.amazon.com/ec2/pricing/).

Lembrando que mesmo usando a versão gratuita, você terá que cadastrar um cartão de crédito internacional.

## Criação da primeira VM

Para iniciar no cloud, você precisa criar uma nova instância de uma VM utilizando uma Amazon Machine Image (AMI). Existem várias imagens disponíveis para Linux e Windows. Imagens pré definidas pela Amazon e outras pela comunidade (Community AMIs). No meu caso escolhi a imagem comunidade [bitnami-tomcatstack](http://bitnami.org/stack/tomcatstack), AMI ID: ami-0eec1167, que vem com o Ubuntu, Tomcat 6, Java 6, Apache e MySQL. As instruções desse post serão baseadas nesse imagem.

Para criar a instância você utilizará um wizard (next, next ... and next) dentro do [AWS Management Console](http://aws.amazon.com/console/). Essa criação possui alguns detalhes que estão comentados abaixo.

Instance tags - Você pode colocar tags na sua instância, "web server", "database", "homologação", "produção", "queridinho" etc. Esses tags fazem muito sentido para quem possui muitas instâncias, sendo muito fácil para aplicar filtros.

Security Group - É o lugar onde você pode liberar as portas do seu servidor, a versão Default vem com tudo bloqueado. No meu caso criei um Security Group novo e liberei as portas de SSH (22) e HTTP (80).

Key pair name - Serve para criar uma chave pública e privada para conectar (SSH) com a sua instância. Faça o download da chave na hora da criação, pois depois não é possível baixar Key pair via o AWS Management Console.

## Endereço da sua instância

O Amazon criará automaticamente um endereço Public DNS para sua instância, que será algo do tipo ec2-11-22-3-444.compute-1.amazonaws.com. Porém se você quiser adicionar um IP, basta ir em Elastic IPs no AWS Management Console, criar um IP e associá-lo a sua instância.

## Acessando a sua instância usando SSH

Basta clicar na sua instância com o botão direito e ir na opção Connect. Será exibido a linha de comando para conectar via SSH. Não se esqueça de aplicar as permissões ```chmod 400``` no arquivo .pem, sem essas permissões o SSH não funcionará. O arquivo .pem é o arquivo que foi baixado na criação do seu Key pair.

> Use chmod to make sure your key file isn't publicly viewable, ssh won't work otherwise:
chmod 400 instancia.pem

Lembrando de alterar o usuário "ubuntu" que aparece na linha de comando para se conectar via SSH, pelo usuário bitnami, que é o usuário padrão criado pela imagem que bitnami.

   De
   ssh -i instancia .pem bitnami @ec2-50-16-0-227.compute-1.amazonaws.com
   Para
   ssh -i instancia .pem bitnami ec2-50-16-0-227.compute-1.amazonaws.com

## Alterando a senha padrão do Tomcat Manager

O usuário e senha padrão para o Tomcat Manager são manager e bitnami respectivamente. Para alterá-los basta acessar o arquivo /opt/bitnami/apache-tomcat/conf/tomcat-users.xml e alterar para usuário e senha mais convenientes.

   sudo vi /opt/bitnami/apache-tomcat/conf/tomcat-users.xml

## Alterando a senha do root do MySQL

O usuário root do MySQL da instalação padrão do bitnami não possui senha, para defina-la basta executar o comando abaixo.

   mysqladmin -u root password NEWPASSWORD

## Criando um usuário e um banco de dados no MySQL

Apesar de trivial, a receita de bolo abaixo é bem útil.

   mysql --user=root
   mysql> create database meu-banco;
   mysql> grant usage on *.* to meu-user@localhost identified by 'minha-senha';
   # Eu preciso de todos os privilégios para executar DDL e DML
   mysql> grant all privileges on meu-banco.* to meu-user@localhost;

## Fazendo o deploy da sua aplicação

A maneira mais simples de fazer o deploy é diretamente pelo Tomcat Manager. Na minha aplicação o arquivo WAR tem por volta de 10MB. A primeira vez que tentei fazer o deploy através do Tomcat Manager não funcionou, tive que repetir o processo pelo menos duas vezes até funcionar.

## Cuidado com o botão Terminate!!!

No AWS Management Console, se você clicar com o botão direito na sua instância, aparecerá a opção Terminate logo acima de Reboot, Stop e Start. Essa opção não é para fazer Stop forçado, essa opção é para apagar a sua instância, você não poderá usá-la novamente. Ela ficará na sua lista de instâncias por algum tempo (no meu caso ficou menos de uma hora), porém logo mais sumirá.

## Conclusão

Com os poucos passos acima a minha aplicação Java está funcionando perfeitamente no EC2. Por enquanto essa experiência está sendo muito boa, muito menos restritiva da que foi com o GAE. Restrições que caracterizam umas das grandes diferenças entre  [IaaS](http://en.wikipedia.org/wiki/Infrastructure_as_a_service#Infrastructure) (EC2) e [PaaS](http://en.wikipedia.org/wiki/Platform_as_a_service) (GAE).

Algo que não posso deixar de levar em consideração, é que com a hospedagem anterior eu tinha um suporte diferenciado, bastava eu abrir um Support Ticket ou Sales Ticket, que em menos de uma hora eu recebia uma resposta. Na Amazon esse tipo de atendimento só está disponível para as contas com suporte Premium através do [AWS Premium Support](http://aws.amazon.com/premiumsupport/). Por outro lado, podemos obter muita informação através de fóruns, blogs (como esse que vos fala) e obviamente na própria documentação da Amazon.
