--- 
layout: post
title: Publicando um web service JAX-WS + Spring MVC 3
tags: 
- JAX-WS
---
Em continuação aos posts [Guia prático do QuickBooks Web Connector com Java](http://pablocantero.com/blog/2010/08/17/guia-pratico-do-quickbooks-web-connector-com-java/) e [SpringSource Tools Suite – E não é que é bom](http://pablocantero.com/blog/2010/08/27/springsource-tools-suite-e-nao-e-que-e-bom/), vou apresentar uma forma fácil de integrar os Web Services gerados com o [wsimport](https://jax-ws.dev.java.net/jax-ws-ea3/docs/wsimport.html) com o Spring MVC 3, criando um projeto do ZERO, utilizando o template "Spring MVC Project" do SpringSource Tools Suite.

##Primeiro passo - Criando um projeto Spring MVC

File -> New -> Other ... -> SpringSource Tools Suite -> Spring Template Project -> Spring MVC Project

![](/images/posts/Screen-shot-2010-08-28-at-6.46.38-PM.png)


##Segundo passo - Criando um Web Services com wsimport

Para criar o Web Service basta acessar o diretório src/main/java e executar o wsimport, conforme o post [Guia prático do QuickBooks Web Connector com Java](http://pablocantero.com/blog/2010/08/17/guia-pratico-do-quickbooks-web-connector-com-java/).

    cd /Users/pablo/workspace/JavaQuickBooksWebConnector/src/
    wsimport http://developer.intuit.com/uploadedFiles/Support/QBWebConnectorSvc.wsdl -s . -p com.cantero.quickbooks.ws

##Terceiro passo - Criando uma classe de implementação

File -> New -> Class

Vamos criar uma nova classe para implementar a interface QBWebConnectorSvcSoap criada anteriormente pelo wsimport. Lembre-se de atualizar (F5) o Eclipse após o wsimport, para ele carregar as classes e interfaces recém geradas pelo wsimport por fora do Eclipse.

![](/images/posts/Screen-shot-2010-08-28-at-6.52.52-PM.png)

Após criada a classe basta adicionar as anotações [JAX-WS](http://jax-ws.java.net/).

    package com.cantero.teste_ws.ws;
    
    import javax.jws.WebService;
    
    @WebService(serviceName = "QBWebConnectorSvcSoapImpl",
     endpointInterface = "com.cantero.teste_ws.ws.QBWebConnectorSvcSoap")
    public class QBWebConnectorSvcSoapImpl implements QBWebConnectorSvcSoap {
      // ...
    }

##Quarto passo - Publicando o Web Service

Adicionando as definições do Web Service gerado no arquivo app-config.xml gerado pelo template Spring MVC Project.

    <bean>
     <property name="baseAddress" value="http://192.168.0.137:9801/" />
    </bean>
    
    <bean id="QBWebConnectorSvcSoapImpl"
     class="com.cantero.teste_ws.ws.QBWebConnectorSvcSoapImpl" />

##Quinta passo - Acessando seu Web Service

Se tudo deu certo após iniciar o seu servidor seu Web Service estará disponível na url abaixo.

[192.168.0.137:9801/QBWebConnectorSvcSoapImpl?wsdl](http://192.168.0.137:9801/QBWebConnectorSvcSoapImpl?wsdl)

##Considereções

Existem outras formas para integrar Web Services com Spring, algumas delas estão disponíveis na própria [documentação do Spring](http://static.springsource.org/spring/docs/3.0.x/spring-framework-reference/html/remoting.html#remoting-web-services-jaxws-export-standalone), que foi da onde eu me baseie para criar esse post.

A integração com [SimpleJaxWsServiceExporter](http://static.springsource.org/spring/docs/2.5.x/api/org/springframework/remoting/jaxws/SimpleJaxWsServiceExporter.html) é semelhante ao EndPoint.publish disponível no Java SE.

    public class Main {
     public static void main(String[] args) {
     Endpoint.publish("http://192.168.0.137:9801/QBWebConnectorSvcSoapImpl",
     new QBWebConnectorSvcSoapImpl());
     }
    }

Fiz um exemplo com o EndPoint.publish no post [Guia prático do QuickBooks Web Connector com Java](http://pablocantero.com/blog/2010/08/17/guia-pratico-do-quickbooks-web-connector-com-java/).

Tem uma [thread no forum do SpringSource](http://forum.springsource.org/showthread.php?t=50814) que questiona se utilizar o SimpleJaxWsServiceExporter pode acarretar em algum problema performance.

##Principais referências

[static.springsource.org/spring/docs/3.0.x/spring-framework-reference/html/remoting.html#remoting-web-services-jaxws-export-standalone](http://static.springsource.org/spring/docs/3.0.x/spring-framework-reference/html/remoting.html#remoting-web-services-jaxws-export-standalone)
