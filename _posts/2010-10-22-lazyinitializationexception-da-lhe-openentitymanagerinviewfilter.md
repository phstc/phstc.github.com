---
layout: post
title: "LazyInitializationException? Dá-lhe OpenEntityManagerInViewFilter"
---

Quem nunca recebeu o "gratificante" erro LazyInitializationException?

> org.hibernate.LazyInitializationException: failed to lazily initialize

Para resolvê-lo basta adicionar o filtro [OpenEntityManagerInViewFilter](http://static.springsource.org/spring/docs/2.5.x/api/org/springframework/orm/jpa/support/OpenEntityManagerInViewFilter.html) no web.xml, que manterá o JPA EntityManager aberto por todo o request permitindo o lazy loading a partir da view.

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
