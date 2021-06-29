---
theme: hexlet
background: cover
# class: 'text-center'
# highlighter: shiki
---

# Чем другие фреймворки лучше Rails
## Кирилл Мокевнин, Хекслет

---
background: center
layout: center
---

# План

* Middlewares (by design)
* Формы и Модели
* Контейнер зависимостей

---
layout: image
image: assets/rails-api-guide.png
---

---
layout: two-cols
background: common
---

## API

<div class="text-sm">

* ActionDispatch::HostAuthorization
* Rack::Sendfile
* ActiveSupport::Cache::Strategy::LocalCache::Middleware
* Rack::Runtime
* ActionDispatch::RequestId
* Rails::Rack::Logger
* ActionDispatch::ShowExceptions
* ActionDispatch::DebugExceptions
* ActionDispatch::ActionableExceptions
* Rack::Head
* Rack::ConditionalGet

</div>

::right::

## Web

<div class="text-sm">

* Rack::Sendfile
* ActiveSupport::Cache::Strategy::LocalCache::Middleware
* Rack::Runtime
* Rack::MethodOverride
* ActionDispatch::RequestId
* Sprockets::Rails::QuietAssets
* Rails::Rack::Logger
* ActionDispatch::ShowExceptions
* WebConsole::Middleware
* ActionDispatch::DebugExceptions
* ActionDispatch::ActionableExceptions
* ActionDispatch::Cookies
* ActionDispatch::Session::CookieStore
* ActionDispatch::Flash
* ActionDispatch::ContentSecurityPolicy::Middleware
* Rack::Head
* Rack::ConditionalGet
* Rack::TempfileReaper

</div>

---
background: common
---

# Управление порядком и количеством

<div class="code-text-xl">

```ruby
# config/application.rb

# Push Rack::BounceFavicon at the bottom
config.middleware.use Rack::BounceFavicon

config.middleware.insert_after ActionDispatch::Executor,
  Lifo::Cache

config.middleware.swap ActionDispatch::ShowExceptions,
  Lifo::ShowExceptions
```

</div>

---
background: center
layout: center
---

# А еще

* before/after/around action
* ApplicationController

---
background: center
layout: center
---

# Phoenix (Elixir)

---
layout: two-cols
---

```elixir
  pipeline :browser do
    plug(:accepts, ["html"])
    plug(:fetch_session)
    plug(HexletBasics.UserManager.Pipeline)
    plug(HexletBasicsWeb.Plugs.AssignCurrentUser)
    plug(HexletBasicsWeb.Plugs.SetLocale)
    plug(:fetch_flash)
    plug(:protect_from_forgery)
    plug(:put_secure_browser_headers)
    plug(HexletBasicsWeb.Plugs.AssignGlobals)
    plug(HexletBasicsWeb.Plugs.SetUrl)
  end

  pipeline :api do
    plug(:accepts, ["json"])
    plug(:fetch_session)
    plug(HexletBasics.UserManager.Pipeline)
    plug(HexletBasicsWeb.Plugs.AssignCurrentUser)
  end

  pipeline :api_for_webhooks do
    plug(:accepts, ["json"])
    plug(:fetch_session)
  end
```

::right::

```elixir
scope "/api", HexletBasicsWeb do
  pipe_through(:api)

  resources "/lessons", Api.LessonController
end

scope "/api/webhooks", HexletBasicsWeb do
  pipe_through(:api_for_webhooks)

  post("/sparkpost/process",
    Api.Webhooks.SparkpostController, :process)
end

scope "/", HexletBasicsWeb do
  pipe_through(:browser)

  get("/", PageController, :index)
  resources("/pages", PageController)
  get("/robots.txt", PageController, :robots)

  resources("/session", SessionController, singleton: true)
  resources "/registrations", UserController
  get("/confirm", UserController, :confirm)
end
```

---
background: common
---

<div class="code-text-xl">

```elixir
defmodule HexletBasicsWeb.Plugs.SetLocale do
  import Plug.Conn

  def init(options), do: options

  def call(conn, _) do
    langs = Application.fetch_env!(:hexlet_basics, :langs)
    locale = Map.get(langs, conn.host, "en")

    Gettext.put_locale(HexletBasicsWeb.Gettext, locale)
    conn
    |> assign(:locale, locale)
  end
end
```

</div>

---
background: common
layout: two-cols-right-full
---

# NodosJS

* Rails
* Django
* Phoenix
* Laravel

::right::

<div class="code-text-xl">

```yaml
pipelines:
  browser:
    - '@nodosjs/view/fetchFlash'
    - example/setCurrentUser
  api:
    - example/setLocale

scopes:
  - path: /api
    pipeline: api
    routes:
      - resources: users
  - path: /
    pipeline: browser
    routes:
      - resources: users
      - resource: session
```

</div>

---
layout: center
background: center
---

# Формы

* strong_params
* form_for (simple_form)

---
layout: center
background: center
---

# Основные задачи

* Массовое присвоение
* Нормализация данных
* Валидация

---
layout: center
background: common
---

# Rails

<div class="code-text-xl">

```ruby
# mass-assigment
attrs = params.require(:user)
              .permit(:email, :password, :password_confirmation)

# normalization
attrs[:email].downcase!

user = User.new attrs

# validation
if user.save
  # ...
end
```

</div>

---
layout: center
background: center
---

# Уровни валидации

* Проверка корректности данных
* Ограничения конкретной формы (бизнес правила)
* Проверка бизнес правил
* Ограничения хранилища

---
layout: center
background: center
---

# Пароль

* Подтверждение (корректность)
* Формат (бизнес-правила)
* Наличие (бизнес-правила по ситуации)

---
layout: center
background: common
---

# Django

<div class="code-text-xl">

```python
from django.forms import ModelForm
from hexlet.models import Article

class ArticleForm(ModelForm):
    headline = forms.CharField(required=True, max_length=100)

    class Meta:
        model = Article
        fields = ['pub_date', 'headline', 'content', 'reporter']
```

</div>

---
layout: center
background: common
---

# Reform

<div class="code-text-xl">

```ruby
class AlbumForm < Reform::Form
  property :title
  validates :title, presence: true

  property :artist do
    property :full_name
    validates :full_name, presence: true
  end

  collection :songs do
    property :name
  end
end
```

</div>

---
layout: center
background: common
---

# ActiveFormModel

<div class="code-text-xl">

```ruby
class UserSignUpForm < User
  include ActiveFormModel

  permit :first_name, :email, :password

  validates :password, presence: true

  def email=(value)
    if value.present?
      write_attribute(:email, value.downcase)
    else
      super
    end
  end
end
```

</div>

---
layout: center
background: center
class: 'text-center'
---

# Зависимости

## Полиморфизм подтипов

---
layout: center
background: common
---

# Внешние системы

<div class="code-text-xl">

```ruby
# Docker
image = Docker::Image.create('fromImage' => 'ubuntu:14.04')

# Facebook Ads
ad_account = FacebookAds::AdAccount.get('id', 'name')

# Consul
foo_service = Diplomat::Service.get('foo')

# Bash
BashRunner.start(command) do |output|
  log << output
end
```

</div>

---
layout: center
background: center
---

# Как тестировать?

---
layout: center
background: center
---

# Варианты

<div class="code-text-xl">

```ruby
# 1. Webmock
stub_request(:post, 'ads.facebook.com').
  with(body: 'abc', headers: { 'Content-Length' => 3 })

# 2. Monkey-patching
class FacebookAds::AdAccount
  # Правим все что хотим
end

# 3. if
if !Rails.env.test?
  ad_account = FacebookAds::AdAccount.get('id', 'name')
end
```

</div>

---
layout: center
background: center
---

# Полиморфизм (подтипов)

* Никакой связи с наследованием
* В Ruby утиная типизация
* Реализуется за счет инверсии зависимостей

---
layout: center
background: center
---

# Полиморфная функция

<div class="code-text-xl">

```ruby
# Например Docker::Service
def some_function(service)
  service = client.get('runner')
  # Тут выполняем нужную логику
end

# Где-то вызов
# Может быть как классом так и объектом
# Зависит от реализации
some_function(Docker::Service)
```

</div>

---
layout: center
background: center
---

# Фейк

<div class="code-text-xl">

```ruby
class DockerApiStub
  def initialize(url, options); end

  attr_reader :connection

  def containers(all); end

  def run_once(image_name, command); end

  def get_info(_cuid)
    { 'Id' => 'asdfasd09f87asdfasdfasdf' }
  end
end
```

</div>

---
layout: center
background: center
---

<div class="code-text-xl">

```ruby
class ApplicationContainer
  extend Dry::Container::Mixin

  if Rails.env.production?
    register :active_campaign { ActivecampaignManager.new 'key' }
    register :sparkpost_client, -> { SimpleSpark::Client.new }
  else
    register :active_campaign, -> { ActivecampaignManagerStub.new }
  end

  if Rails.env.test?
    register :sparkpost_client, -> { SparkpostClientStub.new }
    register(:docker_exercise_api_klass, -> { DockerApiStub })
  else
    register(:docker_exercise_api_klass, -> { DockerApi })
  end
end
```

</div>

---
layout: center
background: center
---

# Использование

<div class="code-text-xl">

```ruby
Import = Dry::AutoInject(ApplicationContainer)

class SomeJob < ApplicationJob
  include Import['sparkpost_client', 'docker_exercise_api_klass']

  def perform(id)
    # Подготовка
    sparkpost_client.do_something()
    docker_exercise_api_klass.foo()
  end
end
```

</div>

---
layout: center
background: center
---

# Куда еще подумать

* Null Object
* Авторизация
* Painless Rails
  * Иерархия контроллеров
  * Иерархия моделей
  * Мутаторы вместо колбеков
  * Сервисный слой (Service Layer)

---
layout: center
background: center
---

# Спасибо
## twitter.com/mokevnin
