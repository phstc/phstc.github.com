--- 
layout: post
title: Configurando o Meta description no WordPress
tags: 
- Meta description
- WordPress
status: publish
type: post
published: true
meta: 
  _edit_last: "1"
  _syntaxhighlighter_encoded: "1"
  dsq_thread_id: "175770077"
---
Nem sei mais o quão importante são as tags de Meta description para indexação de conteúdo hoje em dia. Já vi sites bem posicionados que não usavam tags Meta description.

<!--more-->

Porém no [Webmaster Tools](http://www.google.com/webmasters/tools/) na seção "Google em Diagnostics -> HTML suggestions" o primeiro item e sobre Meta description.

No meu caso todas as páginas estavam com a mesma tag Meta description.

Para resolver esse problema fiz uma pequena alteração no meu template através dos seguintes passos:

1. Acesse "WordPress Admin -> Aparência -> Editor".

2. Selecione o template desejado para fazer a alteração.

3. Selecione o cabeçalho (header.php) no menu da direita.

Comente o atual Meta description se existir.

    <?php
    /*
    <Meta name="description" content="<?php bloginfo('description'); ?>" />
    */
    ?>

Insira o código abaixo para gerar o Meta description baseada na página (post, tag, categoria ou página).

    <?php
    function get_page_meta_description(){
    	$custom_field_meta_description = get_post_meta(get_the_ID(), 'meta_description_field', true);
    	if($custom_field_meta_description != ''){
    		return $custom_field_meta_description;
    	} elseif ( is_single() ) {
    		return get_the_title();
    	} elseif ( is_category() ) {
    		return get_single_cat_title();
    	/*
    	} elseif ( is_tag() ) {
    		return get_single_tag_title(); get single tag não está funcionando
    	*/
    	} elseif ( is_page() ){
    		return get_the_title();
    	} else {
    		return get_bloginfo('description');
    	}
    }
    ?>
    <meta name="description" content="<?php echo get_page_meta_description(); ?>" />

A opção "meta_description_field" serve para definir uma Meta description especifica através um campo personalizado.

![](/images/posts/Screen-shot-2010-10-23-at-2.11.07-PM.png)
