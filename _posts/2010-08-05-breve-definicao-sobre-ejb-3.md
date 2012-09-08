--- 
layout: post
title: "Breve defini\xC3\xA7\xC3\xA3o sobre EJB 3"
tags: 
- EJB 3
- JEE
- JPA
- JSE
- MDB
- session beans
status: publish
type: post
published: true
meta: 
  _edit_last: "1"
  _wp_old_slug: ""
  _syntaxhighlighter_encoded: "1"
  dsq_thread_id: "177379118"
---
EJB 3 classifica os beans em três tipos fundamentais, baseados no que eles fazem:

* Session beans
* Message driven beans
* Entity beans

##Session beans

São beans com o propósito de realizar alguma operação de negócio  especifica, como consultar o histórico de crédito de um cliente.

Pode ser tanto com estado ou sem estado. Um **session bean com estado** é bean típico para um carrinho de compras, onde os itens são adicionados no  carrinho passo a passo, por isso o bean precisa manter o estado. Já um **session bean sem estado** é um bean geralmente usado para operações como um consultar o histórico de crédito de um cliente, ou seja, para operações que não precisam guardar estado

Um session bean pode ser executado local, remoto por [RMI](http://pt.wikipedia.org/wiki/RMI) ou por Web Services. Sendo que somente os session beans sem estado, podem ser expostos como [Web Services](http://pt.wikipedia.org/wiki/Web_service).

* Um session bean não sobrevive a queda ou parada temporário do servidor

* Os sessions beans necessitam de um EJB container para serem executados

##Message driven beans

Assim como os session beans, seu propósito é para realizar operações  de negócio. A diferença e que as operação não recebem resposta automática, são executadas de forma assíncrona

Os MDBs são invocados  através de mensagens enviadas ao servidor de mensagens como IBM WebSphere MQ, SonicMQ, Oracle Advanced Queueing e TIBCO

Um exemplo de sua utilização seria para enviar uma solicitação para reabastecer o estoque de uma loja, que é uma solicitação que não necessita de resposta automática

* Os message drive beans necessitam de um EJB container para serem executados

**Entity beans

Os entities beans, acredito que são os beans mais conhecidos e mais  utilizados, são as conhecidas entitdades JPA

JPA é uma especificação de [ORM](http://en.wikipedia.org/wiki/Object-relational_mapping), onde as implementações mais conhecidas são pelos frameworks [Hibernate](http://pt.wikipedia.org/wiki/Hibernate) e [Oracle TopLink](http://en.wikipedia.org/wiki/TopLink).

Entity bean é a forma que EJB 3 controla a persistência de dados, definindo o padrão para:

* Criação de metadados, mapeamento de entidades para tabelas do modelo relacional
* API EntityManager, API para realização de CRUD (create, retrieve, update e delete)
* Java Persistence Query Language (JPQL), linguagem parece com o SQL ANSI para pesquisa de dados persistidos

Os entities beans não necessitam de um container EJB para serem executados.
