---
layout: post
title: "Cola rápida para criar Cron Jobs"
---

As instruções abaixo são baseados no linux [CentOS](http://en.wikipedia.org/wiki/Centos).

## Motivação

A motivação para escrever esse post, foi para criar um Cron para ser executado de hora em hora no [Heroku](http://heroku.com/), pois o [Add-on Cron](http://addons.heroku.com/cron) do Heroku só é gratuito para uma execução por dia, eu precisava executar de hora em hora.

Para isso criei um Cron que invoca a minha url no Heroku de hora em hora.

    #! /bin/sh
    wget pablocantero.com/service

## Como fazer de graça?

Existem serviços gratuitos na internet para isso, como o [www.mywebcron.com](http://www.mywebcron.com), que podem ser uma opção interessante para invocar URLs de tempos em tempos.

## Crontab

Para criar um Cron você pode utilizar o programa [Crontab](http://pt.wikipedia.org/wiki/Crontab).

    crontab usage: crontab [-u user] file
     crontab [-u user] [ -e | -l | -r ]
     (default operation is replace, per 1003.2)
     -e  (edit user's crontab)
     -l  (list user's crontab)
     -r  (delete user's crontab)
     -i  (prompt before deleting user's crontab)
     -s  (selinux context)

## Editando com o Vi

No CentOS por padrão ao usar o crontab -e, abrirá o [Nano](http://en.wikipedia.org/wiki/Nano_%28text_editor%29), que não estou muito acostumado. Portanto prefiro editar com o [Vi](http://pt.wikipedia.org/wiki/Vi), para isso basta abrir diretamente o arquivo Cron do usuário.

O arquivo Cron do usuário fica no caminho /var/spool/cron/, no meu caso o root.

    vi /var/spool/cron/root

**Diretórios auxiliares

Qualquer script que esteja nos diretórios abaixo será executado de hora em hora, diariamente, semanalmente ou mensalmente, de acordo com o diretório.

* /etc/cron.hourly - De hora em hora
* /etc/cron.daily - Diariamente
* /etc/cron.weekly - Semanalmente
* /etc/cron.monthly - Mensalmente

Essa configuração fica no arquivo /etc/crontab.

    SHELL=/bin/bash
    PATH=/sbin:/bin:/usr/sbin:/usr/bin
    MAILTO=root
    HOME=/

    # run-parts
    21 * * * * root run-parts /etc/cron.hourly
    35 1 * * * root run-parts /etc/cron.daily
    58 5 * * 0 root run-parts /etc/cron.weekly
    23 2 18 * * root run-parts /etc/cron.monthly

## Parâmetros para criar um Cron

A configuração de um Cron é feita com o seguinte padrão:

[minutos] [horas] [dias do mês] [mês] [dias da semana] [usuário] [comando]

Sendo:

Minuto: 0-59
Hora: 0-23
Dia do mês: 1-31
Mês: 1-12
Dia da semana: 0-6, onde domingo = 0, segunda = 1 etc

## Exemplo de Cron

Para executar um script que recupera o [resultado da Mega Sena](http://www1.caixa.gov.br/loterias/loterias/megasena/megasena_resultado.asp) no site Caixa. Precisamos executar um script [Web scraping](http://en.wikipedia.org/wiki/Web_scraping), das 18h até 22h, toda quarta-feira e sábado. Que é quando ocorre os sorteios da Mega Sena.

Para isso teremos que adicionar a seguinte configuração.

    vi /var/spool/cron/root
    # No final do arquivo adicionar
    59 17,18,19,20,21,22 * * 3,6 /meus_scripts/megasena_web_scraping.sh

O megasena_web_scraping.sh será executado 17:59, 18:59, 19:59, 20:59, 21:59 e 22:50, toda quarta-feira = 3 e sábado = 6

Seguindo o [comentário](http://pablocantero.com/blog/2011/01/10/cola-rapida-para-criar-cron-jobs/?preview=true&amp;preview_id=933&amp;preview_nonce=7b5bb8def6#comment-127304159) do [@nadilsons](http://twitter.com/nadilsons), também possível gerenciar os intervalos com 17-22, simplificando o Cron acima.

    59 17-22 * * 3,6 /meus_scripts/megasena_web_scraping.sh

Lembre-se que é importante definir o parâmetro minuto, pois se deixar com *, o script será executado de minuto em minuto.

## Executando em intervalo de minutos

Se somente o parâmetro de minuto estiver configurado, isso não fará que o script seja executado a cada minuto, na verdade ele será executado sempre no minuto configurado no intervalo de hora em hora.

    1 * * * * /meus_scripts/script.sh

Portanto o script acima será executado sempre no minuto 1 de hora em hora. Para fazer com que esse script seja executado a cada um minuto, o intervalo de minutos precisa ser configurado da seguintes formas:

    0-59 * * * * /meus_scripts/script.sh
Ou

    */1 * * * * /meus_scripts/script.sh

Onde */1, indica o intervalo de 1 minuto.
