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
---

# Зависимости
