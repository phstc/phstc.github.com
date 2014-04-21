--- 
layout: post
title: "JavaScript para notificar se um site for alterado"
tags: 
- Bookmarklet
- JavaScript
- Notifier
---
{% include JB/setup %}

Inspirado em uma thread de um grupo de e-mail que participo, onde aguardávamos ansiosamente a abertura das inscrições para o Google Developer Day 2010 (F5, F5, F5 ...). Criei um Bookmarklet para notificar quando um site for alterado.
<!--more-->

##Como usar

1. Clique e arraste <a title="Notificar alterações do site" href="javascript:(function(){/*@author Pablo Cantero - http://pablocantero.com*/var xmlHttp = getXMLHttpObj();if(xmlHttp == null){alert('Failed to load XMLHTTP');return;}var actual = getPageContent(xmlHttp, window.location.href);var intervalId = window.setInterval(function(){var current = getPageContent(xmlHttp, window.location.href);if(actual != current){alert('This page has been modified since you opened it');window.clearInterval(intervalId);}}, 1000);function getPageContent(xmlHttp, url){xmlHttp.open('GET', window.location.href, false);xmlHttp.send('');return xmlHttp.responseText;}function getXMLHttpObj(){if(typeof(XMLHttpRequest)!='undefined')return new XMLHttpRequest();var axO=['Msxml2.XMLHTTP.6.0', 'Msxml2.XMLHTTP.4.0','Msxml2.XMLHTTP.3.0', 'Msxml2.XMLHTTP', 'Microsoft.XMLHTTP'];for(var i = 0; i &lt; axO.length; i++){try{return new ActiveXObject(axO[i]);}catch(e){}}return null;}})()">Notificar Alterações</a> até a barra de favoritos do navegador;

2. Acesse o site que você deseja ser notificado;

3. Clique no link "Notificar Alterações" na barra de favoritos.

Pronto! Toda vez que o site for alterado, o Bookmarklet disparará um alert notificando que houve alterações.

##Código JavaScript

[gist.github.com/751586](https://gist.github.com/751586)
