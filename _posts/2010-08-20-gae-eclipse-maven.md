
---
layout: post
title: "GAE + Eclipse + Maven"
---

Esse é um tutorial para demonstrar como utilizar o [GAE](http://code.google.com/appengine), com [Maven](http://maven.apache.org) ([maven-gae-plugin](http://code.google.com/p/maven-gae-plugin/)) no [Eclipse](http://www.eclipse.org).

O código fonte do guia está disponível no [GitHub](http://github.com/phstc/testapp-gae-maven-eclipse).

## Meu setup

* Mac OSX 10.6.4 x86_64
* Java version: 1.6.0_20
* Eclipse JEE Helios - [Mac OS X)](http://www.eclipse.org/downloads/download.php?file=/technology/epp/downloads/release/helios/R/eclipse-jee-helios-macosx-cocoa-x86_64.tar.gz)
* Apache Maven 2.2.0


## Criação da aplicação com um archetype

```bash
matilde:workspace pablo$ mvn archetype:generate -DarchetypeGroupId=net.kindleit -DarchetypeArtifactId=gae-archetype-jsp -DarchetypeVersion=0.6.0 -DgroupId=com.cantero.testapp_cantero -DartifactId=testapp-cantero -DarchetypeRepository=http://maven-gae-plugin.googlecode.com/svn/repository
[INFO] Scanning for projects...
[INFO] Searching repository for plugin with prefix: 'archetype'.
[INFO] ------------------------------------------------------------------------
[INFO] Building Maven Default Project
[INFO]    task-segment: [archetype:generate] (aggregator-style)
[INFO] ------------------------------------------------------------------------
[INFO] Preparing archetype:generate
[INFO] No goals needed for project - skipping
[INFO] [archetype:generate {execution: default-cli}]
[INFO] Generating project in Interactive mode
[INFO] Archetype defined by properties
[INFO] Using property: groupId = com.cantero.testapp_cantero
[INFO] Using property: artifactId = testapp_cantero
Define value for property 'version': 1.0-SNAPSHOT:
[INFO] Using property: package = com.cantero.testapp_cantero
[INFO] Using property: package = com.cantero.testapp_cantero
Define value for property 'gaeApplicationName': : testapp-cantero
[INFO] Using property: gaeApplicationVersion = 0
Define value for property 'gaePluginVersion': : 0.6.0
[INFO] Using property: gaeVersion = 1.3.5
Confirm properties configuration:
groupId: com.cantero.testapp_cantero
artifactId: testapp_cantero
version: 1.0-SNAPSHOT
package: com.cantero.testapp_cantero
package: com.cantero.testapp_cantero
gaeApplicationName: testapp_cantero
gaeApplicationVersion: 0
gaePluginVersion: 0.6.0
gaeVersion: 1.3.5
Y: Y
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESSFUL
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 1 minute 56 seconds
[INFO] Finished at: Thu Aug 19 20:25:20 BRT 2010
[INFO] Final Memory: 16M/81M
[INFO] ------------------------------------------------------------------------
```

O nome quer for informado no `gaeApplicationName`, deve ser um nome válido de aplicação no GAE, ele será posteriormente usado no deploy.

Para o `gaePluginVersion` foi utilizada a versão 0.6.0.
As versões estão disponíveis no [repositório](http://maven-gae-plugin.googlecode.com/svn/repository/net/kindleit/gae-archetype-jsp/).

## Adicionando Eclipse capabilities

``bash
mvn eclipse:eclipse


matilde:workspace pablo$ cd testapp-cantero/
matilde:testapp-cantero pablo$ mvn eclipse:eclipse
[INFO] Scanning for projects...
[INFO] ------------------------------------------------------------------------
[ERROR] FATAL ERROR
[INFO] ------------------------------------------------------------------------
[INFO] Error building POM (may not be this project's POM).

Project ID: com.cantero.testapp_cantero:testapp-cantero
POM Location: /Users/pablo/workspace/testapp-cantero/pom.xml

Reason: Resolving expression: '${name}': Detected the following recursive expression cycle: [name] for project com.cantero.testapp_cantero:testapp_cantero at /Users/pablo/workspace/testapp-cantero/pom.xml
```

Caso aparece esse erro, bastar alterar o pom.xml do projeto

```xml
<name>testapp-cantero</name>
<description>Aplicação teste GAE + Eclipse + Maven</description>
```

Voltando a executar `mvn eclipse:eclipse` deve retornar BUILD SUCCESSFUL.

## Importando o projeto para o Eclipse

Para importar o projeto para o Eclipse, bastar ir em File -> Import -> Existing Projects into Workspace -> [selecionar o diretório do projeto] e importar.

Quando eu importei o projeto para o Eclipse, tinha algumas classes padrões criadas pelo archetype, mas elas estavam com packages errados, bastou corrigir os packages e os imports que o projeto ficou sem erros.

Não esquecendo também de corrigir o caminho do `IndexServlet` no `web.xml` para o package selecionado.

```xml
<servlet>
  <servlet-name>IndexServlet</servlet-name>
  <servlet-class>com.cantero.testapp_cantero.web.IndexServlet</servlet-class>
</servlet>
```

## Primeiro deploy no GAE

Na primeira vez é necessário alterar o código do pom.xml

De:

`<gae.version>1.3.5</gae.version>`

Para:

`<gae.version>1.3.1</gae.version>`

A versão 1.3.5 não está disponível (até a data desse post no [repositório oficial do maven](http://mvnrepository.com/artifact/com.google.appengine/appengine-java-sdk))

Para o primeiro deploy é necessário executar:


```bash
$ mvn gae:unpack
$ mvn gae:deploy
```

Se tudo deu certo, sua aplicação estará disponível no GAE.