---
layout: post
title: "Olha quem está falando"
description: ""
category: 
tags: [Twilio, CaaS]
---
{% include JB/setup %}

![Olha quem está falando](/assets/images/posts/look-who-is-talking.jpg)

# Senta que lá vem a história

Antes de começar a falar do [Twilio](http://www.twilio.com/), vou falar a minha motivação para usá-lo. Foi para um aplicativo de suporte por telefone, o [Support Roulette](https://github.com/phstc/support-roulette). No final, acabamos não usando exatamente o Support Roulette, mas foi uma ótima base. 

Devido à facilidade de implementar uma aplicação que pode responder uma chamada telefônica, gravar uma mensagem de voz, enviar e receber SMS, ou seja, uma aplicação que fala, me inspirei para escrever esse post e inclusive apresentar um [Tech Talk](https://github.com/phstc/support-roulette/tree/master/slides) (Vim talk rs) sobre o assunto.

## Support Roulette

No trabalho, precisamos de um telefone de suporte 24x7 caso o site fique indisponível e para outras possíveis incidências urgentes.

Nossa proposta inicial foi nomear um desenvolvedor por semana como suporte.

O [MVP](http://en.wikipedia.org/wiki/Minimum_viable_product) foi comprar um celular baratinho (a bateria do danado é infinita, mas também, não tem nem o [jogo da cobrinha](http://en.wikipedia.org/wiki/Snake_(video_game\))) e deixá-lo com o suporte da vez. A ideia do celular foi principalmente para o nosso atendimento poder ligar em um número fixo "Suporte Site", não precisando buscar quem é o desenvolvedor de plantão, pegar o número do celular e ligar.

### Problemas com o celular baratinho

Todo mundo já tem um celular. Carregar um segundo é um tanto quanto incômodo. Fácil, fácil de esquecer.

### Solução

Nossa solução foi usar algum serviço de "Communication as a Service" (CaaS).

Usando um serviço de CaaS, podemos registrar um número fixo no Brasil, que, ao receber uma chamada telefônica, a redireciona para o número do celular do desenvolvedor de plantão.

### Voip

Os serviços convencionais de Voip que avaliamos, apesar de permitirem a criação de número fixo no Brasil e inclusive permitirem o redirecionamento de ligação, não permitem que alteremos o número de redirecionamento via API. Teríamos que acessar um painel de controle semanalmente para configurar o novo número de redirecionamento. Isso é inviável pois somos programadores, queremos agendar as escalas de suporte e ter um sistema que automaticamente avise o desenvolvedor de plantão e faça o redirecionamento. 

## Por que Twilio?

* Não precisa de contrato
* Não tem valor mínimo de serviço
* Não tem taxa de adesão
* É Pré pago, não tem mensalidade
* Tem API simples e bem documentada

### Outras opções

Existem [outras opções](http://en.wikipedia.org/wiki/Twilio#Competitors) de CaaS, porém nem todas disponibilizam número no Brasil ou possuem API simples e bem documentada. Das que eu avaliei, o [Plivio](http://www.plivo.com/) pareceu o concorrente mais forte.

## Apps que se comunicam

Concedendo poderes de comunicação (voz e SMS) para uma aplicação, podemos adicionar diversos recursos, como:

* Alertas e notificações
  * Mensagem de falta ou renovação de estoque
  * Notificação de site indisponível. Dá para "facilmente" implementar um [Pingdom](https://www.pingdom.com/) caseiro com o [Monit](http://mmonit.com/monit/) + Twilio
* Promoções
  * Códigos de promoções por SMS
  * Perguntas premiadas "Envie um SMS para … e concorra …"
* Lembretes
  * Notificações de agendas e eventos
* Questionários
  * Pesquisas de satisfação que podem ser respondidas tanto por SMS quanto por voz. Dá para facilmente implementar uma [URA](http://pt.wikipedia.org/wiki/Unidade_de_resposta_aud%C3%ADvel), capturando as respostas pelos números digitados
* Administrações de sistemas
  * Obter diagnósticos do sistema por SMS ou voz
  * Executar operação no sistema por comando de voz ou SMS
* Automação de vendas
  * Enviar SMS para requisitar ou consultar estoque de produtos
* Sistema de identificação de usuário
  * Validar o usuário por SMS ou voz
* Bike Sampa
  * No [Bike Sampa](http://www.bikesampa.com.br/) seria bem interessante se ao invés de ter que instalar um aplicativo e precisar de 3G, pudéssemos cadastrar nosso celular, enviar um SMS com número do ponto e bicicleta e liberá-la, sem tráfego 3G, sem aplicativo… até o celular baratinho sem o jogo da cobrinha poderia ser usado
* Support Roulette ;)

No site do Twilio tem "[Customer Success Stories](http://www.twilio.com/gallery/customers)", com alguns casos de uso de empresas que utilizam o Twilio.

### Pingdom caseiro com Monit + Twilio

O Monit é um sistema para gerenciamento e monitoramento de processo, programas etc.

Pingdom é um serviço para monitoramento de uptime, downtime e performance de websites.

Um caso de uso do Pingdom padrão é usá-lo somente para notificar se o site ficou indisponível. De uma forma caseira é possível fazer isso com o Monit e Pingdom.

#### Monit script

    # /etc/monit.d/website.conf
    
    check process website with pidfile /var/run/website.pid
	    start program = "/etc/init.d/website start"
	    stop program  = "/etc/init.d/website stop"
	    if failed host 127.0.0.1 port 3000 protocol http with timeout 30 seconds then exec "/bin/bash -c 'ruby /usr/bin/notify-site-is-down.rb'"

#### Ruby script

    # /usr/bin/notify-site-is-down.rb
    require "rubygems"
    require "twilio-ruby"
    
    account_sid = "ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
    auth_token = "yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy"
    
    @client = Twilio::REST::Client.new account_sid, auth_token
    
    @client.account.SMS.messages.create(
      from: "+5511…",
      to: "+5511…",
      body: "Houston we have a problem, the website is down."
    )

### Custos

No caso do Support Roulette há dois custos: um de voz, quando o Twilio recebe uma ligação e redireciona para o desenvolvedor de plantão e outro de SMS, para notificá-lo que ele é o plantonista da vez. 

#### Voz

O custo para redirecionamento de voz é de duas pernas, é cobrado para receber e também para fazer (redirecionar) a ligação.


Para receber uma ligação: 1¢/minuto

Para fazer uma ligação: 33¢/minuto

#### SMS

> SMS messaging from an American Twilio phone number to Brazil starts at 1.2¢ however this price will vary depending on the carrier you are sending SMS messages to.
> Elaine Tsai, Twilio Support

Número fixo no Brasil ainda não envia SMS. Para enviar um SMS Tupiniquim é necessário ter um número americano. O principal inconveniente em usar um número americano para enviar SMS é no caso de resposta, ter que responder para um número americano.

Para enviar um SMS: a partir de 1.2¢/mensagem

#### Custo número fixo

Número fixo brasileiro = 3$/mês

Número fixo americano = 1$/mês

## Começando com o Twilio

Os passos iniciais do Twilio são: [TwiMLTM: the Twilio Markup Language](http://www.twilio.com/docs/api/twiml) e
[Twilio REST Web Service Interface](http://www.twilio.com/docs/api/rest).

Exemplo de TwiMLTM:

    <?xml version="1.0" encoding="UTF-8"?>
    <Response>
        <Say voice="woman">Please leave a message after the tone.</Say>
    </Response>

Recomendo também uma olhada no [Support Roulete](https://github.com/phstc/support-roulette), pois ele é uma aplicação [Sinatra](https://github.com/sinatra/sinatra) super simples, que usa a [gem twilio-ruby](https://github.com/twilio/twilio-ruby) e [builders](https://github.com/phstc/support-roulette/blob/master/views/support_roulette_call.builder) para gerar TwiMLTM, podendo ser hospedada gratuitamente no Heroku.

## Twimlets

[Twimlets](https://www.twilio.com/labs/twimlets) são aplicações prontas que implementam funcionalidades básicas, como [redirecionamento de ligação](https://www.twilio.com/labs/twimlets/forward), [caixa postal](https://www.twilio.com/labs/twimlets/voicemail) etc. Se você quiser hoje mesmo ter um número fixo em São Paulo que redirecione para um celular do Acre, basta  criar uma conta no Twilio, configurar um twimlet de redirecionemto e, em menos de 10 minutos, estará tudo funcionado. 
