---
layout: post
title: "Guia prático do QuickBooks Web Connector com Java"
---

O [QuickBooks Web Connector](http://marketplace.intuit.com/webconnector) é uma ótima alternativa para fazer integrações com o [QuickBooks](http://quickbooks.intuit.com), pois com QBWC não é necessário utilizar os componentes COM+, que limitam a escolha da tecnologia para ser utilizado na integração.

O código fonte do guia está disponível no GitHub:

[github.com/phstc/JavaQuickBooksWebConnector](http://github.com/phstc/JavaQuickBooksWebConnector).

## Vantagens do QuickBooks Web Connector

* A comunicação com o seu servidor e realizada através de Web services, dessa forma não é necessário atrelar sua aplicação com componentes COM+ (windows).
* Os Web services podem ser desenvolvidos em qualquer linguagem e em qualquer plataforma, desde de que tenha suporte para Web services.
* O QBWC pode ser configurado para invocar seu servidor de tempos em tempos, portanto não é necessário que o usuário faça uma intervenção manual, tendo que abrir um aplicativo, clicar em sincronizar etc. Basta configurar o QBWC para invocar o Web services no intervalo de tempo desejado.
* É mais fácil para criar aplicações genéricas (SaaS), pois a cada requisição do QBWC, além de usuário e senha, ele envia identificação da aplicação, portanto um Web services pode atender a diversos clientes.

## Ambiente

Para desenvolver a integração com Java, estou usando:

* Eclipse
* QuickBooks Enterprise Edition 10.0
* QuickBooks Web Connector 2.0.0.139

## Primeiro passo - Criação do arquivo QWC

O arquivo QWC é o arquivo de configuração do seu Web services no QBWC.

```xml
<?xml version="1.0"?>
<QBWCXML>
  <AppName>WCWebService1</AppName>
  <AppID></AppID>
  <AppURL>http://localhost/WCWebService/WCWebService.asmx</AppURL>
  <AppDescription>A short description for WCWebService1</AppDescription>
  <AppSupport>http://developer.intuit.com</AppSupport>
  <UserName>iqbal1</UserName>
  <OwnerID>{57F3B9B1-86F1-4fcc-B1EE-566DE1813D20}</OwnerID>
  <FileID>{90A44FB5-33D9-4815-AC85-BC87A7E7D1EB}</FileID>
  <QBType>QBFS</QBType>
  <Scheduler>
    <RunEveryNMinutes>2</RunEveryNMinutes>
  </Scheduler>
</QBWCXML>
```

## Segundo passo - wsimport QBWebConnectorSvc.wsdl

Para construir um Web services a partir do WSDL será utilizado o [wsimport](http://download.oracle.com/docs/cd/E17802_01/webservices/webservices/docs/2.0/jaxws/wsimport.html), que gerará o Web Services JAX-WS compatível.

O WSDL para o Web Services está disponível na url [developer.intuit.com/uploadedFiles/Support/QBWebConnectorSvc.wsdl](http://developer.intuit.com/uploadedFiles/Support/QBWebConnectorSvc.wsdl).

Para iniciar criei um projeto Java Básico com o Eclipse "File -> New -> Java Project -> JavaQuickBooksWebConnector".

```bash
cd /Users/pablo/workspace/JavaQuickBooksWebConnector/src/

wsimport http://developer.intuit.com/uploadedFiles/Support/QBWebConnectorSvc.wsdl -s . -p com.cantero.quickbooks.ws
```

Parâmetros

* -s para especificar o diretório onde será gerando o código fonte
* -p para especificar o package

## Terceiro passo - Criando uma classe de implementação

A definição dos métodos exigidos pela interface QBWebConnectorSvcSoap estão definidos no [QuickBooks Web Connector Programmer's Guide](http://developer.intuit.com/qbsdk-current/doc/pdf/qbwc_proguide.pdf).

A linguagem para requisições do QBWC é [qbXML](http://developer.intuit.com/qbSDK-Current/Common/newOSR/index.html).

### Classe de implementação

```java
package com.cantero.quickbooks.ws;

import java.util.ArrayList;

import javax.jws.WebService;

/*
* http://developer.intuit.com/qbsdk-current/doc/pdf/qbwc_proguide.pdf
*/
@WebService(endpointInterface = "com.cantero.ws.client.QBWebConnectorSvcSoap")
public class ItemQueryRqSoapImpl implements QBWebConnectorSvcSoap {

  @Override
  public ArrayOfString authenticate(String strUserName, String strPassword) {
    ArrayOfString arr = new ArrayOfString();
    arr.string = new ArrayList<String>();
    arr.string.add("The first element is a token for the web connector’s session");
    arr.string.add(""); //To use the currently open company, specify an empty string
    return arr;
  }

  @Override
  public String closeConnection(String ticket) {
    // TODO Auto-generated method stub
    return null;
  }

  @Override
  public String connectionError(String ticket, String hresult, String message) {
    // TODO Auto-generated method stub
    return null;
  }

  @Override
  public String getLastError(String ticket) {
    // TODO Auto-generated method stub
    return null;
  }

  /*
  * A positive integer less than 100 represents the percentage of work completed. A value of 1 means one percent complete, a value of 100 means 100 percent complete--there is no more work. A negative value means an error has occurred and the Web Connector responds to this with a getLastError call. The negative value could be used as a custom error code.
  */
  @Override
  public int receiveResponseXML(String ticket, String response,
  String hresult, String message) {
    // a value of 100 means 100 percent complete--there is no more work
    return 100;
  }

  @Override
  public String sendRequestXML(String ticket, String strHCPResponse,
  String strCompanyFileName, String qbXMLCountry, int qbXMLMajorVers,
  int qbXMLMinorVers) {
    // Example qbXML to Query for an Item
    // http://www.consolibyte.com/wiki/doku.php/quickbooks_qbxml_itemquery
    String query = "<?xml version=\"1.0\" encoding=\"utf-8\"?><?qbxml version=\"7.0\"?><QBXML><QBXMLMsgsRq onError=\"stopOnError\"><ItemQueryRq requestID=\"SXRlbVF1ZXJ5fDEyMA==\"><OwnerID>0</OwnerID></ItemQueryRq></QBXMLMsgsRq></QBXML>";
    return query;
  }
}
```

A instrução `<OwnerID>0</OwnerID>` é utilizada para que o QuickBooks retorne os Custom Fields. Na versão QuickBooks Enterprise Edition 10, pode ser inserido mais de 5 custom fields, [mas até o momento só é possível recuperar os 5 primeiros utilizando qbXML](https://idnforums.intuit.com/messageview.aspx?catid=7&amp;threadid=13998).

## Quarto passo - Importando o QWC no QBWC

Basta ir ao QuickBooks Web Connector "Add an application -> Selecionar o QWC -> Autorizar acesso no QuickBooks".

## Quinto passo - Testando o Web services

```java
package com.cantero.quickbooks.ws;

import javax.xml.ws.Endpoint;

public class Main {
  public static void main(String[] args) {
    Endpoint.publish("http://192.168.0.137:54321/ItemQueryRqSoapImpl",
    new ItemQueryRqSoapImpl());
  }
}
```

Para executar o Main diretamente pelo Eclipse basta ir em "Run As -> Java Application, Debug As -> Java Application".

Se tudo deu certo a url [http://192.168.0.137:54321/ItemQueryRqSoapImpl?wsdl](http://192.168.0.137:54321/ItemQueryRqSoapImpl?wsdl) deverá exibir o seu wsdl.

## Principais referências

* [marketplace.intuit.com/webconnector](http://marketplace.intuit.com/webconnector)
* [QuickBooks Web Connector Programmer's Guide](http://developer.intuit.com/qbsdk-current/doc/pdf/qbwc_proguide.pdf)
* [Referência para qbXML](http://developer.intuit.com/qbSDK-Current/Common/newOSR/index.html)
