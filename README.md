# zachestnyibiznes-webintegration-ruby
Инструкция WEB интеграции на Ruby on Rails

## Установка и настройка интеграции

Добавьте в Gemfile гем rest-client:

```ruby
# Gemfile

gem 'rest-client'
```

Выполните установку гема, запустив bundle:

```bash
$ bundle install
```

После этого можно вставить приложение код интеграции:

Ваш action в контроллере:

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

Ваш view для этого экшена:

```erb
<%= raw @response %>
```

## Рекомендации и примечания



