--- 
layout: post
title: "PropertyPlaceholderConfigurer - Usando properties nas configura\xC3\xA7\xC3\xB5es do Spring"
tags: 
- properties
- Spring
status: publish
type: post
published: true
meta: 
  _syntaxhighlighter_encoded: "1"
  _edit_last: "1"
  _wp_old_slug: propertyplaceholderconfigurer-usando-properties-nas-configuracoes-do-sprin
  dsq_thread_id: "175591229"
---
Os arquivos properties são muito úteis para configurar variáveis de ambiente etc. Quem nunca usou um properties?!?! Com o Spring [PropertyPlaceholderConfigurer](http://static.springsource.org/spring/docs/2.0.x/api/org/springframework/beans/factory/config/PropertyPlaceholderConfigurer.html), fica muito fácil! :)

Esse post é um guia rápido de utilização do Spring PropertyPlaceholderConfigurer.

##Criando um properties

Utilizei a seguinte estrutura ```src/resources/app-config.properties```.

    # WS Config
    ws.baseAddress=http://127.0.0.1:9801/
    # DB Config
    db.url=jdbc:mysql://localhost:3306/db
    db.username=username
    db.password=password
    db.driverClass=com.mysql.jdbc.Driver

##Configurando o app-config.xml

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

Lembrando que o app-config.xml é o nome do arquivo de configuração do Spring, que podem ser vários e com nomes diferentes dependendo da aplicação.

    <servlet>
    	<servlet-name>Spring MVC Dispatcher Servlet</servlet-name>
    	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    	<init-param>
    		<param-name>contextConfigLocation</param-name>
    		<param-value>/WEB-INF/spring/*.xml</param-value>
    	</init-param>
    	<load-on-startup>1</load-on-startup>
    </servlet>

##Utilizando múltiplos properties

Caso sua aplicação utilize múltiplos properties, basta fazer o seguinte ajuste na declação do bean PropertyPlaceholderConfigurer.

    <bean id="propertiesPlacholder" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer" lazy-init="false">
    	<property name="locations">
    		<list>
    			<value>classpath:/resources/app-config-db.properties</value>
    			<value>classpath:/resources/app-config-ws.properties</value>
    		</list>
    	</property>
    </bean>

##Outra forma de como usar

    <context:property-placeholder location="classpath:/resources/app-config-db.properties" />

##Principais referências

* [stackoverflow.com/questions/534199/how-to-collect-spring-properties-from-multiple-files-for-use-on-a-single-bean](http://stackoverflow.com/questions/534199/how-to-collect-spring-properties-from-multiple-files-for-use-on-a-single-bean)
* [static.springsource.org/spring/docs/2.0.x/api/org/springframework/beans/factory/config/PropertyPlaceholderConfigurer.html](http://static.springsource.org/spring/docs/2.0.x/api/org/springframework/beans/factory/config/PropertyPlaceholderConfigurer.html)
