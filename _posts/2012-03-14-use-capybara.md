---
layout: post
title: "Use Capybara!"
---

Aplicações não podem falhar, um usuário não pode clicar em algo e receber um erro. Principalmente em aplicações web abertas. Imaginem uma busca no Google retornando "500 Internal Server Error".

O Twitter para evitar possíveis erros usava "twitter is over capacity". Não é excelente, mas é melhor do que retornar um "503 Service Unavailable".

##Minizando erros

Uma forma para minizar esses são com [Testes de aceitação](http://en.wikipedia.org/wiki/Acceptance_testing), que também são conhecidos como Testes funcionais ou Testes de caixa-preta, estes testes podem ajudar muito para garantir que sua aplicação tenha o comportamento esperado.

[Capybara](https://github.com/jnicklas/capybara) é um framework de Testes de aceitação para aplicações web. Com ele é possível testar que dado o input A a aplicação deve retornar o output B. Mas precisamente com Capybara se submeto o form de login informando o usuário e senha válidos a aplicação deve me redirecionar para a página de usuário logado.

##Teste no Controller sem Capybara

Com [Rails](http://rubyonrails.org/) e [RSpec](https://github.com/rspec/rspec) é bem fácil para os controllers, porém não testa as views no máximo se elas estão renderizando corretamente se usar o  [render_views](http://rubydoc.info/gems/rspec-rails/file/README.md#render_views).

O generator padrão do Rails cria automáticamente testes para os controllers.

    describe PessoasController do
      describe "GET 'index'" do
        it "returns http success" do
          get 'index'
          response.should be_success
        end
      end
    end

Já ajuda bastante. Pelo menos garantirá que não tem nenhum erro no método index. Porém não testa o input e output.

Adicionando mais assertions.

    describe PessoasController do
      describe "GET 'index'" do
        it "deve filtrar por nome" do
          pablo = Factory :pessoa, nome: "Pablo"
          Factory :pessoa, nome: "Megan Fox"
          get 'index', nome: "Pablo"
          assigns[:pessoas].should eq [pablo]
          response.should be_success
        end
      end
    end

Já melhora um pouco, já testa se o controller adicionou o Array de pessoas no request.

## Teste com Capybara

Com o Capybara além de testar o controller, testa a view, inclusive com Javascript. É um pouco diferente do teste acima, pois é um teste caixa-preta, não vai testar se existe algum atributo no request, aliás o request não está nem disponível no contexto do Capybara.

Teste de login usando Capybara.

    describe "Login com sucesso", :type => :request do
      it "login com sucesso deve redirecionar para a página novas corridas" do
        password = "123123123"
        user = Factory.create :user, password: password
        visit new_user_session_path
        fill_in "user[email]",                 with: user.email
        fill_in "user[password]",              with: password
        click_button "user_submit"
        current_path.should == corridas_proximas_path
      end
    end

    describe "Login sem sucesso", :type => :request do
      it "login sem sucesso deve voltar para a página de login" do
        visit new_user_session_path
        fill_in "user[email]",                 with: "nonono"
        fill_in "user[password]",              with: "nonono"
        click_button "user_submit"
        current_path.should == new_user_session_path
      end
    end

##Cucumber

Testes de aceitação baseados em ferramentas como o [Cucumber](http://cukes.info) ([BDD](http://pt.wikipedia.org/wiki/Behavior_Driven_Development)) também são uma alternativa para testar funcionalidades.

Eu particularmente acho mais prático e rápido escrever testes com Capybara do que com o Cucumber, porém se as [User Stories](http://dannorth.net/whats-in-a-story/) já fizeram parte do processo e vierem em um formato que seja fácil para copiar, colar e executar, talvez seja mais interessante usar o Cucumber.
