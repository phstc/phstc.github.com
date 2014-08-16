---
layout: post
title: "Spring MVC 3 + MongoDB - Parte 2"
---

Em continuação ao [Spring MVC 3  + MongoDB - Parte 1](http://pablocantero.com/blog/2010/09/03/spring-mvc-3-mongodb-parte-1), vou me basear na utilização do DbFactoryBean do post [Plain Simple MongoDB Spring Integration](http://java.dzone.com/articles/plain-simple-mongodb-spring), com algumas observações adicionais.

## Baixando o MongoDB

O MongoDB usado para escrever esse post foi OS X 64-bit - Versão 1.6.2, disponível na páginas de [downloads](http://www.mongodb.org/downloads) do MongoDB.

Baixei e salvei no meu workspace.

    $ /Users/pablo/workspace/mongodb-osx-x86_64-1.6.2/

## Driver para o MongoDB

Eu não achei, mas também não procurei muito um repositório Maven para o driver do MongoDB, portanto eu simplesmente baixei o [mongo-2.1.jar](http://github.com/downloads/mongodb/mongo-java-driver/mongo-2.1.jar) e salvei no diretório ```WEB-INF/lib```.

    mkdir workspace/spring-mongodb/src/main/webapp/WEB-INF/lib
    cd workspace/spring-mongodb/src/main/webapp/WEB-INF/lib
    wget http://github.com/downloads/mongodb/mongo-java-driver/mongo-2.1.jar

Após baixar e atualizar F5 o projeto no Eclipse.

![](/images/posts/Screen-shot-2010-09-03-at-12.58.17-PM.png)

## Adicionando o mongo-2.1.jar no Build Path

Botão direito no projeto -> Propriedades -> Java Build Path -> Libraries -> Add JARs...

![](/images/posts/Screen-shot-2010-09-03-at-12.57.17-PM.png)

## Configurando app-config.xml

Para configurar o app-config.xml, basta adicionar as seguintes linhas.

    <bean id="propertiesPlacholder"
     class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer"
     lazy-init="false">
     <property name="location" value="classpath:/resources/app-config.properties" />
     </bean>

     <context:property-placeholder location="classpath:db.properties" />
     <bean id="mongo">
     <constructor-arg value="${db.host}" />
     <constructor-arg value="${db.port}" />
     </bean>
     <bean id="db">
     <property name="mongo" ref="mongo" />
     <property name="name" value="${app.db.name}" />
     </bean>

## #app-config.properties

    db.host=localhost
    #porta default
    db.port=27017
    app.db.name=spring-mongodb-app

No post [PropertyPlaceholderConfigurer – Usando properties nas configurações do Spring](http://pablocantero.com/blog/2010/09/03/propertyplaceholderconfigurer-usando-properties-nas-configuracoes-do-sprin/) tem mais informações sobre Spring e arquivos properties.

## Classe DbFactory

    package com.cantero.spring_mongodb.dao;

    import org.springframework.beans.factory.FactoryBean;
    import org.springframework.beans.factory.annotation.Required;
    import org.springframework.util.Assert;

    import com.mongodb.DB;
    import com.mongodb.Mongo;

    public class DbFactoryBean implements FactoryBean<DB> {

     private Mongo mongo;
     private String name;

     @Override
     public DB getObject() throws Exception {
     Assert.notNull(mongo);
     Assert.notNull(name);
     return mongo.getDB(name);
     }

     @Override
     public Class<?> getObjectType() {
     return DB.class;
     }

     @Override
     public boolean isSingleton() {
     return true;
     }

     @Required
     public void setMongo(Mongo mongo) {
     this.mongo = mongo;
     }

     @Required
     public void setName(String name) {
     this.name = name;
     }

    }

## Relembrando Factory

[Factory Method](http://en.wikipedia.org/wiki/Factory_method_pattern) e [Abstract Factory](http://en.wikipedia.org/wiki/Abstract_factory_pattern) são listados no [GoF](http://c2.com/cgi/wiki?GangOfFour) como patterns creacionais, que lidam com a instanciação de objetos.

Factory Method provê uma classe de decisão, na qual retorna uma das muitas possíveis subclasses de uma classe base abstrata, dependendo do dado fornecido.

    CarroAbstract carro = CarroFactory.get("Vectra");

Abstract Factory provê uma interface para se criar e retornar uma de muitas famílias de objetos relacionados.

    FabricaAbstract fabrica = AbstractFabricaFactory.getInstance("Chevrolet");
    CarroAbstract carro = fabrica.get("Vectra");

## Singleton/Prototype

O FactoryBean pode ser [Spring Singleton](http://static.springsource.org/spring/docs/2.0.x/reference/beans.html#beans-factory-scopes-singleton) (uma única instância para todas as requisições) ou [Spring Prototype](http://static.springsource.org/spring/docs/2.0.x/reference/beans.html#beans-factory-scopes-prototype) (uma nova instancia para cada requisição), lembrando que apesar do nome, são implementações essecialmente iguais, mas implementadas de forma diferente do que os respectivos patterns no GoF.

Singleton é o escopo padrão para criação de Beans no Spring, que pode ser alterado na declaração do Bean.

## Voltando... BaseDAOImpl

    <bean id="baseDAOImpl" class="com.cantero.spring_mongodb.dao.BaseDAOImpl">
     <property name="db" ref="dbFactory"/>
    </bean>

## #BaseDAOImpl.java

    package com.cantero.spring_mongodb.dao;

    import com.mongodb.DB;

    public class BaseDAOImpl implements IBaseDAO {

     //private Logger logger = org.slf4j.LoggerFactory
     //        .getLogger(BaseDAOImpl.class);

     protected DB db;

     public void setDb(DB db) {
     this.db = db;
     }

     protected DB getDb() {
     return db;
     }
    }

## Utilizado o BaseDAOImpl nos Services

    package com.cantero.spring_mongodb.service;

    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Service;

    import com.cantero.spring_mongodb.dao.IBaseDAO;

    @Service
    public class PessoaServiceImpl implements IPessoaService {

     @Autowired
     IBaseDAO baseDAO;

    }

## No Controller

    package com.cantero.spring_mongodb.controller;

    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Controller;
    import com.cantero.spring_mongodb.service.IPessoaService;

    @Controller
    public class PessoaController {

     @Autowired
     IPessoaService pessoaService;

    }

## Iniciando o MongoDB

A primeira vez que for iniciar o MongoDB lembre-se de criar o diretório /data/db

    $ mkdir /data/db
    $ cd workspace/mongodb-osx-x86_64-1.6.2/bin/
    $ ./mongod

Continuará...
