---
layout: post
title: "Por que eu troquei o WordPress pelo Jekyll?"
---

Eu já estava utilizando o [WordPress](http://wordpress.org/) (WP) desde o meu primeiro post que foi em [22/07/2010](http://pablocantero.com/blog/2010/07/22/guia-rapido-de-configuracao-do-cucumber/). Não tenho muitas críticas ao WP, ele funciona bem. O principal motivo para migrar para o  [Jekyll](https://github.com/mojombo/jekyll) foi pela simplicidade e redução custos.

## Jekyll

Jekyll é um gerador de blog estático, você cria seus posts em [Markdown](http://pt.wikipedia.org/wiki/Markdown) e ele converte para HTML. É estático, portanto não usa banco de dados ou conteúdo dinâmico.

```bash
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
```

Ele é muito simples, porém exige um certo conhecimento técnico. Recomendo para pessoas que saibam instalar uma [Ruby gem](http://rubygems.org/), que conheçam [Git](https://en.wikipedia.org/wiki/Git_(software)) e Markdown.

Você não precisa necessariamente conhecer Git, porém se conhecer poderá [hospedar no GitHub gratuitamente](https://help.github.com/articles/using-jekyll-with-pages/).

A página [jekyllbootstrap.com](http://jekyllbootstrap.com/) ensina os passos para hospedar no GitHub.

> Zero to Hosted Jekyll Blog in 3 Minutes

## Markdown

Usar Markdown facilita muito a vida, é muito simples e prático, dá para editar em qualquer editor de texto. Usar o [WYSIWYG](http://pt.wikipedia.org/wiki/WYSIWYG) do WP, sempre foi muito traumático para mim. O modo preview sempre dá problema, quebra HTML e gera código horrível, ainda mais se usar algum plugin para syntax highlighting.

    Com Jekyll basta eu colocar quatro espaços antes do código
    que ele gerará um HTML do tipo:
		`<pre><code>...</code></pre>`
    Que posso colocar o style (CSS) que eu quiser.

Mais sobre [Markdown](http://daringfireball.net/projects/markdown/syntax).

## Versionamento

Eu uso o Jekyll com Git, portanto tenho o versionamento das minhas alterações. Se quero restaurar ou comparar as versões de um post é muito simples.

```bash

git log
commit fe6d22281b53129b554a26b4645c8bbd340df6fe
Author: Pablo Cantero <pablo@pablocantero.com>
Date:   Wed Mar 14 10:09:13 2012 -0300

   Quebra de linha por css
...

git diff fe6d22281b53129b554a26b4645c8bbd340df6fe

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
```

## Migração

Eu segui os passos de migração da [documentação oficial](https://github.com/mojombo/jekyll/wiki/Blog-Migrations), gerei um WordPress export file e importei para o Jekyll.

```bash
$ ruby -rubygems -e 'require "jekyll/migrators/wordpressdotcom";Jekyll::WordpressDotCom.process("pablocantero.com.xml")'
```

Funciona, porém ele importa o conteúdo em HTML. O Jekyll também funciona com HTML, se o arquivo for `.md` ele interpretará como Markdown, se for `.html` como HTML.

Porém eu queria Markdown, portanto tive que editar manualmente todos os meus posts para convertê-los.

## Redução de custos

Agora eu consigo hospedar gratuitamente no GitHub. No EC2 na região mais barata, com a instância mais barata (on demand) eu gastava aproximadamente [$37 por mês](http://calculator.s3.amazonaws.com/calc5.html).

Claro, não tem comparação uma coisa com a outra, no EC2 eu tinha um servidor, porém para o meu blog o [GitHub Pages](https://pages.github.com/) é mais do que suficiente.

## Domínio próprio

Por padrão no GitHub o site/blog ficará disponível em SEU-USUÁRIO.github.com, para utilizar com domínio próprio basta seguir a [documentação oficial do GitHub Pages sobre custom domains](http://pages.github.com/#custom_domains).