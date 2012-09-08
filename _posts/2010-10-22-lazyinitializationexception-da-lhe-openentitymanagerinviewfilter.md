--- 
layout: post
title: "LazyInitializationException? Dá-lhe OpenEntityManagerInViewFilter"
tags: 
- Java
- Spring
- OpenEntityManagerInViewFilter
type: post
published: true
meta: 
  _syntaxhighlighter_encoded: "1"
  _edit_last: "1"
---
Quem nunca recebeu o "gratificante" erro LazyInitializationException?

> org.hibernate.LazyInitializationException: failed to lazily initialize

Para solucionar basta adicionar o filtro [OpenEntityManagerInViewFilter](http://static.springsource.org/spring/docs/2.5.x/api/org/springframework/orm/jpa/support/OpenEntityManagerInViewFilter.html) no web.xml, que irá manter o JPA EntityManager por todo o request permitindo o lazy loading na view.

    <!-- web.xml -->
    <filter>
        <filter-name>Spring OpenEntityManagerInViewFilter</filter-name>
        <filter-class>org.springframework.orm.jpa.support.OpenEntityManagerInViewFilter</filter-class>
        <init-param>
            <param-name>singleSession</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>Spring OpenEntityManagerInViewFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
