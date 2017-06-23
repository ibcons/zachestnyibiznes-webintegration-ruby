# Инструкция WEB-интеграции API на Ruby on Rails

## Установка и настройка интеграции

* Добавьте в Gemfile гем rest-client:

```ruby
# Gemfile

gem 'rest-client'
```

* Выполните установку гема, запустив bundle:

```bash
$ bundle install
```

После этого можно вставить приложение код интеграции.

* Ваш action в контроллере:

```ruby
#app/controllers/welcome_controller.rb

class WelcomeController < ApplicationController
  def index    
    api_parameters = {} #хеш параметров для запроса

    if params[:id].present?
      api_parameters[:id] = params[:id]
      url = params[:fs].present? ? 'https://zachestnyibiznesapi.ru/v3/data/fs' : 'https://zachestnyibiznesapi.ru/v3/data/card'
    elsif params[:q].present?
      url = 'https://zachestnyibiznesapi.ru/v3/data'
      api_parameters[:string] = params[:q]
    else
      api_parameters[:string] = ''
      url = 'https://zachestnyibiznesapi.ru/v3/data'
    end

    api_parameters[:api_key] = 'ВАШ КЛЮЧ API интеграции'
    api_parameters[:ip_client] = request.ip    

    res = RestClient.post url, api_parameters
    @response = res.body

  rescue RestClient::ExceptionWithResponse => e
     @response = e.response
  end
end
```

* Ваш view для этого экшена:

```erb
<%# app/views/welcome/index.html.erb %>

<%= raw @response %>
```

Мы используем raw-хелпер, так как API интеграции возвращает данные html и их нужно отобразить без санирования разметки.

* После этого запустив экшен http://yourdomain/welcome/ или http://yourdomain/welcome/index, Вы увидите главный экран api интеграции, а приложение будет реагировать на GET-запросы api через форму или вручную.

Если этого не произошло, проверьте маршруты к контроллеру и экшену, например

```ruby
#config/routes.rb

Rails.application.routes.draw do
  get 'welcome/index' 
  #...другие маршруты 
end
```

## Рекомендации и примечания
1. Название контроллера welcome_controller и экшена index выбраны произвольно. Они могут быть какими угодно, не влияет на интеграцию

2. В коде контроллера перехватывается исключение RestClient::ExceptionWithResponse. Это служит лишь для того, чтобы разработчик при возникновении проблем смог вывести в view переменную @response и изучить ее вывод.

3. Разметка формы для поиска делается по усмотрению разработчика. Позднее она будет добавлена в API интеграции.

4. Вы можете для интеграции использовать не только гем [rest-client](https://github.com/rest-client/rest-client) но и другие его аналоги, например [faraday](https://github.com/lostisland/faraday) или [httparty](https://github.com/jnunemaker/httparty). Делайте это, если точно понимаете код интеграции, его потребуется изменить если Вы выберете другой гем вместо [rest-client](https://github.com/rest-client/rest-client). 

5. Опционально! Ваш личный ключ API интеграции можно вставлять прямо в код как в примере:

```ruby
api_parameters[:api_key] = 'ВАШ КЛЮЧ API интеграции'
```
однако это не совсем желательно. Ключ может попасть в публичный репозиторий вместе с Вашим кодом приложения. Чтобы лучше защитить ключ API, Вы можете:

* Если у Вас версия Rails 5.1 и выше, используйте для хранения ключа [Encrypted Secrets](http://edgeguides.rubyonrails.org/5_1_release_notes.html#encrypted-secrets). 
* Если версия меньше 5.1 используйте гем [dotenv-rails](https://github.com/bkeepers/dotenv) или переменные окружения самого сервера. Так же есть возможность при Rails 4.1 и выше, использовать [возможность secrets.yml](http://rusrails.ru/4_1_release_notes#config-secrets-yml), исключив сам файл секретов из git репозитория

## Демонстрационная площадка

У интеграции API "За честный бизнес" c Ruby on Rails существует [демонстрационная площадка](http://chestbiz.herokuapp.com/), какая обновляется и обслуживается специалистом Rails. Вы можете запускать ее, слать ей запросы, позднее она будет изменяться по мере изменения API интеграции, как и данный документ.
