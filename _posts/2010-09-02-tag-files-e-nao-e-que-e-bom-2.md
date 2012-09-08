--- 
layout: post
title: "Tag Files - E n\xC3\xA3o \xC3\xA9 que \xC3\xA9 bom"
tags: 
- Java
- JSP
- JSTL
- tag files
status: publish
type: post
published: true
meta: 
  _edit_last: "1"
  _wp_old_slug: tag-files-e-nao-e-que-bom-2
  _syntaxhighlighter_encoded: "1"
  dsq_thread_id: "175638280"
---
Tag Files são uma forma rápida de encapsular conteúdos em páginas JSP. Elas são muito "mais fáceis" de implementar que uma tag library, mesmo que seja uma [javax.servlet.jsp.tagext.SimpleTagSupport](http://download-llnw.oracle.com/javaee/1.4/api/javax/servlet/jsp/tagext/SimpleTagSupport.html).

Como disse um [grande amigo meu](http://blogs.abril.com.br/java-cabeca):

> Tag File é um JSP com mais poderes

Vou utilizar um exemplo real, que eu precisava exibir a data de sincronização de um sistema legado com o web services, inicialmente eu estava exibindo a data da última sincronização, sempre que sincronizava eu atribuía um new Date para o atributo lastSync. O problema é o timezone do servidor era diferente do que o do cliente. Para resolver o problema optei para exibir a diferença em minutos, horas e dias da última sincronização.

##Tag File DateDiff

Para armazenar as Tag Files eu criei o seguinte diretório WEB-INF/tags/, sendo que qualquer arquivo .tag, que for salvo nesse diretório será uma nova Tag File.

##WEB-INF/tags/dateDiff.tag

    <%@ tag import="java.util.Date" body-content="empty"%>
    <%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
    <%@ attribute name="date1" required="true" rtexprvalue="true" type="java.util.Date"%>
    <%@ attribute name="date2" required="false" rtexprvalue="true" type="java.util.Date"%>
    <%
    Date date1 = (Date) jspContext.getAttribute("date1");
    Date date2 = (Date) jspContext.getAttribute("date2");
    if (date2 == null) {
    date2 = new Date();
    }
    int diffMinutes = (int) ((date2.getTime() - date1.getTime()) / 1000) / 60;
    int diffHours = diffMinutes / 60;
    int diffDays = diffHours / 24;
    String dateDiffMsg = "";
    if (diffMinutes < 60) {
    dateDiffMsg = diffMinutes + " minutos";
    } else if (diffHours < 24) {
    dateDiffMsg = diffHours + " horas";
    } else {
    dateDiffMsg = diffDays + " dias";
    }
    jspContext.setAttribute("dateDiffMsg", dateDiffMsg);
    %>
    ${dateDiffMsg}

(discussões a parte sobre utilizar scriptlets ou não)

##Para utilizar a Tag File dateDiff.tag

Todos os .tag salvos em WEB-INF/tags podem ser acessados como métodos após a inclusão da diretiva taglib.

    <%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
    <%@ taglib prefix="fn" uri="http://java.sun.com/jsp/jstl/functions"%>
    <%@ taglib tagdir="/WEB-INF/tags" prefix="h" %>
    <html>
    <head>
    ...
    <table id="box-table-a">
    <thead>
    <tr>
    <th>Name</th>
    <th>Full Name</th>
    <th>Quantity On Hand</th>
    <th>Quantity On SalesOrder</th>
    <th>Quantity On Order</th>
    <th>Última Sincronização</th>
    </thead>
    <tfoot>
    <tr>
    <td colspan="5">Total de registros</td>
    <td>${fn:length(requestScope.itemInventoryRetList)}</td>
    </tr>
    </tfoot>
    <tbody>
    <c:forEach items="${requestScope.itemInventoryRetList}"
    var="itemInventoryRet">
    <tr>
    <td>${itemInventoryRet.name}</td>
    <td>${itemInventoryRet.fullName}</td>
    <td>${itemInventoryRet.quantityOnHand}</td>
    <td>${itemInventoryRet.quantityOnSalesOrder}</td>
    <td>${itemInventoryRet.quantityOnOrder}</td>
    <td>
    <h:dateDiff date1="${itemInventoryRet.lastQuickBooksSync}"/>
    </td>
    </tr>
    </c:forEach>
    </tbody>
    </table>
    ...

##Resultado

![](/images/posts/Screen-shot-2010-09-02-at-1.48.02-AM.png)

