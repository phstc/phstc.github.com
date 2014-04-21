---
layout: post
title: "Por que eu troquei o WordPress pelo Jekyll?"
tags: [Jekyll, WordPress]
---
{% include JB/setup %}

Eu já estava utilizando o [WordPress](http://wordpress.org/) (WP) desde o meu primeiro post que foi em [22/07/2010](http://pablocantero.com/blog/2010/07/22/guia-rapido-de-configuracao-do-cucumber/). Não tenho muitas críticas ao WP, ele funciona bem. O principal motivo para migrar para o  [Jekyll](https://github.com/mojombo/jekyll) foi pela simplicidade e redução custos.

##Jekyll

Jekyll é um gerador de blog estático, onde você cria seus posts em [Markdown](http://pt.wikipedia.org/wiki/Markdown) e eles são convertidos para HTML. É estático, ele não usa banco de dados ou conteúdo dinâmico gerado em runtime.

    rake post title="Hello World!"
    Creating new post: _posts/2012-03-14-hello-world.md

    # _posts/2012-03-14-hello-world.md
    ---
    layout: post
    title: "Hello Word"
    category:
    tags: []
    ---
    Post content.

Ele é muito simples, porém exige conhecimento técnico. Recomendo para pessoas que saibam baixar uma [gem](http://rubygems.org/) e que conheçam Git e Markdown.

Você não precisa necessariamente conhecer Git, porém se conhecer poderá hospedar no GitHub gratuitamente.

A página [jekyllbootstrap.com](http://jekyllbootstrap.com/) ensina os passos para hospedar no GitHub.

> Zero to Hosted Jekyll Blog in 3 Minutes

##Sites que utilizam Jekyll

[github.com/mojombo/jekyll/wiki/Sites](https://github.com/mojombo/jekyll/wiki/Sites)

O primeiro site dessa lista é do [Tom Preston (Mojombo)](http://tom.preston-werner.com/) que é Cofounder do GitHub e autor do Jekyll.

##Markdown

Usar Markdown facilita muito a vida, é muito simples e prático, edito o post diretamente [TextMate](http://macromates.com/) e com syntax highlight para Markdown. Usar o [WYSIWYG](http://pt.wikipedia.org/wiki/WYSIWYG) do WP, sempre foi muito traumático para mim, se usar no modo preview sempre dá problema, quebra HTML e gera código horrível, ainda mais se usar algum plugin para **Syntax Highlighter**, que é muito comum para blogs técnicos.

    Com Jekyll basta eu colocar quatro espaços antes do código
    que ele gerará um HTML do tipo:
    <pre><code>...</code></pre>
    Que posso colocar o style (CSS) que eu quiser.

Mais sobre a [sintaxe do Markdown](http://daringfireball.net/projects/markdown/syntax).

##Versionamento

Eu uso o Jekyll com git, portanto tenho o versionamento dos commits. Se quero restaurar ou comparar uma versão anterior do post é muito simples.

    $ git log
    commit fe6d22281b53129b554a26b4645c8bbd340df6fe
    Author: Pablo Cantero <pablo@pablocantero.com>
    Date:   Wed Mar 14 10:09:13 2012 -0300

        Quebra de linha por css
    ...

    $ git diff fe6d22281b53129b554a26b4645c8bbd340df6fe

    diff --git a/assets/themes/tom/css/screen.css b/assets/themes/tom/css/screen.css
    index bc975ca..bfd514d 100644
    --- a/assets/themes/tom/css/screen.css
    +++ b/assets/themes/tom/css/screen.css
    @@ -174,6 +174,7 @@ ul.posts {
         background-color: #eef;
         font-size: 85%;
         padding: 0 .2em;
    +    word-break: break-word
       }

         #post pre code {

##Migração

Eu segui os passos de migração da [documentação oficial](https://github.com/mojombo/jekyll/wiki/Blog-Migrations), gerei um WordPress export file e importei para o Jekyll.

    $ ruby -rubygems -e 'require "jekyll/migrators/wordpressdotcom";Jekyll::WordpressDotCom.process("pablocantero.com.xml")'

Funciona, porém ele importa o conteúdo em HTML. É isso mesmo, o Jekyll também funciona com HTML, se o arquivo for .md ele interpretará como Markdown, se for .html como HTML.

worém eu queria Markdown, editei post a post que eu tinha, mesmo que muitos já estavam totalmente obsoletos, tecnologia voa... Para mim sempre é muito bom reler coisas antigas e principalmente melhorá-las. Embora para quem não queira fazer isso ou que seja inviável pela quantidade de conteúdo, é possível fazer um search&replace com regex que resolverá boa parte da migração.

##Redução de custos

Agora eu hospedo gratuitamente no GitHub. No EC2 na região mais barata, com a instância mais barata (on demand) eu gastava aproximadamente [$37 por mês](http://calculator.s3.amazonaws.com/calc5.html).

Claro, não tem comparação uma coisa com a outra, no EC2 eu tinha um servidor é diferente, porém só para o meu Blog hospedar GitHub está mais do que bom!

##Domínio próprio

Por padrão no GitHub o site/blog ficará disponível em SEU-USUÁRIO.github.com, para utilizar com domínio próprio basta seguir a [documentação oficial do GitHub Pages sobre custom domains](http://pages.github.com/#custom_domains). Que basicamente terá endereço IP para ser configurado como registro tipo A no gerenciador de DNS do domínio.
