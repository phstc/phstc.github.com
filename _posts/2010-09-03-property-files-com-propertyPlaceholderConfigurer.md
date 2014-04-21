---
layout: post
title: "Property files com PropertyPlaceholderConfigurer"
tags: [Spring, PropertyPlaceholderConfigurer]
---
{% include JB/setup %}

Os arquivos de propriedades são muito úteis para configurar variáveis de ambiente, parâmetros de inicialização etc. Quem nunca usou um [.properties](http://en.wikipedia.org/wiki/.properties)?!

Esse post é um guia rápido de utilização do Spring [PropertyPlaceholderConfigurer](http://static.springsource.org/spring/docs/2.0.x/api/org/springframework/beans/factory/config/PropertyPlaceholderConfigurer.html).

## Criando um properties

Utilizei a seguinte estrutura ```src/resources/app-config.properties```.

    # WS Config
    ws.baseAddress=http://127.0.0.1:9801/
    # DB Config
    db.url=jdbc:mysql://localhost:3306/db
    db.username=username
    db.password=password
    db.driverClass=com.mysql.jdbc.Driver

## Configurando o app-config.xml

    <bean id="propertiesPlacholder" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer" lazy-init="false">
    	<property name="location" value="classpath:/resources/app-config.properties" />
    </bean>
    <bean>
    	<property name="baseAddress" value="${ws.baseAddress}" />
    </bean>
    <!-- DataSource SQL Server -->
    <bean id="dataSource" destroy-method="close" scope="singleton">
    	<property name="driverClassName" value="#{db.driverClass}" />
    	<property name="url" value="${db.url}" />
    	<property name="username" value="${db.username}" />
    	<property name="password" value="${db.password}" />
    </bean>


O app-config.xml é o nome do arquivo de configuração do Spring, que pode mudar de acordo com o seu ambiente.

Exemplo de configuração do app-config.xml.

    <servlet>
    	<servlet-name>Spring MVC Dispatcher Servlet</servlet-name>
    	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    	<init-param>
    		<param-name>contextConfigLocation</param-name>
    		<param-value>/WEB-INF/spring/app-config.xml</param-value>
    	</init-param>
    	<load-on-startup>1</load-on-startup>
    </servlet>


## Utilizando múltiplos properties

Caso sua aplicação utilize múltiplos properties, basta adicioná-los na declaração do bean PropertyPlaceholderConfigurer.

    <bean id="propertiesPlacholder" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer" lazy-init="false">
    	<property name="locations">
    		<list>
    			<value>classpath:/resources/app-config-db.properties</value>
    			<value>classpath:/resources/app-config-ws.properties</value>
    		</list>
    	</property>
    </bean>

## Principais referências

* [stackoverflow.com/questions/534199/how-to-collect-spring-properties-from-multiple-files-for-use-on-a-single-bean](http://stackoverflow.com/questions/534199/how-to-collect-spring-properties-from-multiple-files-for-use-on-a-single-bean)
* [static.springsource.org/spring/docs/2.0.x/api/org/springframework/beans/factory/config/PropertyPlaceholderConfigurer.html](http://static.springsource.org/spring/docs/2.0.x/api/org/springframework/beans/factory/config/PropertyPlaceholderConfigurer.html)
