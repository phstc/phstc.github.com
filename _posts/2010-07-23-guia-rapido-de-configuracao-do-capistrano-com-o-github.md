--- 
layout: post
title: "Guia r\xC3\xA1pido de configura\xC3\xA7\xC3\xA3o do Capistrano com o GitHub"
tags: 
- Capistrano
- deployment
- Git
- GitHub
- Rails
status: publish
type: post
published: true
meta: 
  _wp_old_slug: capistrano-e-github-quicksetup
  _edit_last: "1"
  _syntaxhighlighter_encoded: "1"
  dsq_thread_id: "176475813"
---
Esse é um guia rápido de configuração do [Capistrano](http://www.capify.org/) com o [GitHub](http://github.com).

Os passos desses post foram utilizados no projeto [click-anywhere-to-leave-a-message.com](http://click-anywhere-to-leave-a-message.com), disponível para download no [GitHub](http://github.com/phstc/click-anywhere-to-leave-a-message.com).

##Criando um repositório no GitHub

Após logado no GitHub, no seu [dashboard](https://github.com) aparecerá seus repositórios e um botão para criar um novo repositório.

![](/images/posts/Screen-shot-2010-07-22-at-3.47.04-PM.png)

Após criado o repositório, o GitHub mostrará as instruções para o seu primeiro commit.

     cd click-anywhere-to-leave-a-message.com
     git init
     touch README
     git add README
     git commit -m 'first commit'
     git remote add origin git@github.com:phstc/click-anywhere-to-leave-a-message.com.git
     git push origin master

## .gitignore

Para não compartilhar alguns arquivos no repositório, como usuário/senha do banco de dados, logs e arquivos temporários, basta criar um arquivo .gitignore na raiz do projeto.

    cat .gitignore
    .DS_Store
    *~
    *.swp
    tmp/*
    log/*
    backup/*
    config/database.yml
    Capfile
    config/deploy.rb
    db/*.sqlite3
    mkmf.log
    db/schema.rb

##Instalando e adicionando o Capistrano ao seu projeto

    cd click-anywhere-to-leave-a-message.com
    gem install capistrano
    capify .

##Configurando o deploy.rb

Após adicionar o Capistrano ao seu projeto, basta configurar o arquivo deploy.rb.

    set :application, 'sua aplicacao'
    set :user, 'seu usuario ssh'
    set :password, 'sua senha shh'
    set :domain, 'seu dominio'
    set :mongrel_port, 'sua porta'
    set :mongrel_nodes, "4"
    set :rails_env,     "production"
    set :server_hostname, 'seu hostname'
    set :git_account, 'seu usuario do github'
    set :scm_passphrase,  Proc.new { Capistrano::CLI.password_prompt('sua senha') }
    role :web, 'seu dominio'
    role :app, 'seu dominio'
    role :db, 'seu dominio', :primary => true
    default_run_options[:pty] = true
    set :repository,  "git@github.com:#{git_account}/#{application}.git"
    set :scm, "git"
    set :user, user
    ssh_options[:forward_agent] = true
    set :branch, "master"
    set :deploy_via, :checkout
    set :git_shallow_clone, 1
    set :git_enable_submodules, 1
    set :use_sudo, false
    set :deploy_to, "/home/#{user}/rails_apps/#{application}"

##Adicionado a public key no GitHub

Se aparecerem problemas como Permission denied (publickey).

    servers: ["click-anywhere-to-leave-a-message.com"]
     [click-anywhere-to-leave-a-message.com] executing command
     ** [click-anywhere-to-leave-a-message.com :: out] Permission denied (publickey).
     ** [click-anywhere-to-leave-a-message.com :: out] fatal: The remote end hung up unexpectedly

Quer dizer que você tem que adicionar uma public_key do seu servidor e computador local no GitHub. Para isso, o GitHub disponibiliza tutoriais para [linux](http://help.github.com/linux-key-setup/), [mac](http://help.github.com/mac-key-setup/) e [windows](http://help.github.com/msysgit-key-setup/).

Após gerada a public_key, basta acessar [ssh_bucket](https://github.com/account#ssh_bucket) para incluí-la.

![](/images/posts/Screen-shot-2010-07-22-at-8.42.59-PM.png)

##Fazendo o deploy com o Capistrano

Para fazer o deploy basta executar a instrução cap deploy.

    cd click-anywhere-to-leave-a-message.com
    cap deploy

Se não aparecer nenhum erro (será???), seu Capistrano estará funcionando perfeitamente, no meu caso apareceu o seguinte erro:

    current/script/process/reaper: No such file or directory

Para corrigir eu tive que adicionar as tasks de start, stop e restart no deploy.rb.

    namespace :deploy do
     task :start, :roles => :app do
     run "cd #{current_path} && mongrel_rails start -e production -p #{mongrel_port} -d"
     end
     task :restart, :roles => :app do
     run "cd #{current_path} && mongrel_rails restart"
     end
     task :stop, :roles => :app do
     run "cd #{current_path} && mongrel_rails stop"
     end
    end

##Criando link simbólico para arquivos não compartilhados no repositório

Meu database.yml e outros arquivos estão somente no servidor e não no repositório, portanto sempre após o deploy com o Capistrano, eu crio um link simbólico para esses arquivos.

    after "deploy:finalize_update", "deploy:symlink2"
    #...
    namespace :deploy do
    #...
    desc "Make symlink for config files"
     task :symlink2 do
     run "ln -nsf /home/#{user}/rails_apps/#{application}/database.yml #{release_path}/config/database.yml"
     end
    end

##Diretório log

O Capistrano cria automáticamente um link simbólico para o diretório de log do projeto apontando para /home/#{user}/rails_apps/#{application}/shared/log. Como ele não cria automáticamente o diretório só o link simbólico, precisamos cria-lo manualmente.

    mkdir /home/#{user}/rails_apps/#{application}/shared/
    mkdir /home/#{user}/rails_apps/#{application}/shared/log

##Principais referências

* [Capistrano site oficial](http://www.capify.org)
* [Capistrano com GIT, Tutorial Básico](http://blog.areacriacoes.com.br/2008/6/25/capistrano-com-git-tutorial-b-sico)
* [GitHub Troubleshooting SSH issues](http://help.github.com/troubleshooting-ssh/)
* [Thread no Google Groups sobre o current/script/process/reaper: No such file or directory](http://groups.google.com/group/capistrano/browse_thread/thread/158118e51477a4f9)
* [Capistrano After](http://www.capify.org/index.php/After#See_Also)
