--- 
layout: post
title: "Ler excel com Java usando o JExcelApi"
tags: 
- Java
- JExcelApi
- POI
---
{% include JB/setup %}

Esse post é uma continuação do post [Upload com Spring MVC](http://pablocantero.com/blog/2010/09/29/upload-com-spring-mvc), pois além do upload eu precisava ler o Excel e adicionar as informações no Banco de Dados.

Para ler Excel estou usando o [JExcelApi](http://jexcelapi.sourceforge.net/). Outra opção seria o [Apache POI](http://poi.apache.org/) que é mais popular.

##Primeiro passo

Adicionar a dependência JExcelApi no pom.xml.

    <properties>
      <jexcelapi.version>2.6</jexcelapi.version>
    </properties>
    <dependency>
      <groupId>net.sourceforge.jexcelapi</groupId>
      <artifactId>jxl</artifactId>
      <version>${jexcelapi.version}</version>
    </dependency>

##Segundo passo

Escrever o código para ler o Excel. No caso estou usando a action de upload do artigo [Upload com Spring MVC](http://pablocantero.com/blog/2010/09/29/upload-com-spring-mvc/).

    @RequestMapping(value = "upload", method = RequestMethod.POST)
     public String upload(HttpServletRequest request)
     throws BiffException, IOException {
     MultipartHttpServletRequest multipartRequest = (MultipartHttpServletRequest) request;
     MultipartFile multipartFile = multipartRequest.getFile("file");
     Workbook workbook = Workbook
     .getWorkbook(multipartFile.getInputStream());
    
     Sheet sheet = workbook.getSheet("Nome da planilha");
    
     for (int i = 0; i < sheet.getRows(); i++) {
    for(int j = 0; j < sheet.getColumns(); j++){
    Cell celulaJ = sheet.getCell(j, i);
    System.out
    .println("Conteúdo da célula " + j + ": " + celularJ.getContents());
    }
     }
    
     return "redirect:upload-success";
     }

##Principais referências

[oficinadanet.com.br/artigo/java/lendo_planilhas_excel_com_java](http://www.oficinadanet.com.br/artigo/java/lendo_planilhas_excel_com_java)
