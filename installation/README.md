# Devise 젬의 설치 및 셋업

아래와 같이 `devise` 젬을 셋업한다.

```bash
$ rails g devise:install
      create  config/initializers/devise.rb
      create  config/locales/devise.en.yml
=================================================================

Some setup you must do manually if you haven't yet:

  1. Ensure you have defined default url options in your environments files. Here is an example of default_url_options appropriate for a development environment in config/environments/development.rb:

       config.action_mailer.default_url_options = { host: 'localhost:3000' }

     In production, :host should be set to the actual host of your application.

  2. Ensure you have defined root_url to *something* in your config/routes.rb.
     For example:

       root to: "home#index"

  3. Ensure you have flash messages in app/views/layouts/application.html.erb.
     For example:

       <p class="notice"><%= notice %></p>
       <p class="alert"><%= alert %></p>

  4. If you are deploying on Heroku with Rails 3.2 only, you may want to set:

       config.assets.initialize_on_precompile = false

     On config/application.rb forcing your application to not access the DB
     or load models when precompiling your assets.

  5. You can copy Devise views (for customization) to your app by running:

       rails g devise:views
```

실행 결과에서 보는 바와 같이 추가 조치사항에 대한 안내에 따라 차례대로 하나씩 작업을 한다.

먼저 `config/environments/development.rb` 파일을 열고 하단에 아래와 같이 추가한다.

```ruby
Rails.application.configure do

...중략...

  config.action_mailer.default_url_options = { host: 'localhost:3000' }

end
```

이 옵션은 `devise` 젬을 이용할 경우 회원가입시 안내 이메일을 보낼 때 필요한 것이다.

다음은, 루트 URL을 지정하기 위해서 우선 `welcome` 컨트롤러와 `index` 액션을 생성한다.

```bash
$ rails g controller welcome index
```

그리고, `config/routes.rb` 파일을 열고 아래와 같이 보이는데,

```ruby
Rails.application.routes.draw do
  get 'welcome/index'
end
```

루트를 지정하기 위해 아래와 같이 수정한다. 위의 안내문에서는 `root to: "home#index"`와 같이 되어 있으나 `routes.rb` 파일 내의 커맨트 내용을 참고하면 `to:`를 생략해도 된다.

```ruby
Rails.application.routes.draw do
  root 'welcome#index'
end
```

> #### Caution::주의
> 
> 주의할 것은 `welcome/index`가 아니라 `welcome#index`와 같이 중간에 해시 기호(`#`)를 사용해야 한다.

이제 터미널 커맨드라인 쉘에서 아래와 같이 로컬 웹서버를 구동하고
```bash
$ rails s
```

터미널 창을 하나 더 생성하여 프로젝트 디렉토리로 이동한 후
아래와 같이 명령을 실행하면 손쉽게 브라우저를 열어 볼 수 있다.

```bash
$ open http://localhost:3000
```

`http://localhost:3000`로 접속하면 처음에 보이던 `Welcome aboard` 페이지 대신에 `welcome` 컨트롤러의 `index` 액션의 뷰 템플릿 페이지가 보이게 된다.

> #### Caution::주의
> 
> 젬을 새로추가하거나 어플리케이션의 설정 파일을 수정한 경우에는 반드시 서버를 재실행해야 제대로 반영된다.

[루트를 지정하지 않은 상태의 웹페이지 화면]
![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/auth_blog/2014-05-28_11-08-51_zpsc7fdf050.png)

[루트를 지정한 후의 웹페이지 화면]
![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/auth_blog/2014-05-28_11-11-55_zpsc9073f5d.png)

실제 브라우저에서 보이는 내용은 위의 화면과는 다소 차이가 있다. `body` 태그 영역에 표시되는 내용의 상단에 `60px` 정도의 공백이 보이게 된다. 이것은 나중에 `Bootstrap`의 `navbar` 클래스를 `div` 태그에  적용하여 메뉴를 추가할 때를 고려하여 미리 공간을 확보해 놓은 것이다.

이제 세번째 조치사항으로 어플리케이션 레이아웃 파일(`app/views/layouts/application.html.erb`)을 열어 `<head></head>` 태그내에 아래와 같이 메타 정보를 추가한다.

```html
<meta charset="utf-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0" />
```

마지막의 `viewport` 메타 정보는 디바이스별로 동적인 레이아웃을 제대로 보이도록 하기 위해서 화면의 스케일을 보정하기 위한 것이다.
또한, `flash` 메시지(주로 컨트롤로 액션의 처리 결과 정보를 전달하기 위해 사용하는 해시 데이터임)가 보이도록 코드를 추가한다.

```html
<body>
  <p class="notice"><%= notice %></p>
  <p class="alert"><%= alert %></p>

  <%= yield %>

</body>
```

네번째 조치사항은 레일스 3.2 로 만든 프로젝트를 허로쿠로 배포할 때에 해당하기 때문에 여기서는 생략하도록 한다.(이 샘플 프로젝트는 레일스 4.1버전으로 작성하기 때문에 신경쓰지 않아도 된다.)

이제 `devise` 젬에서 필요로하는 사용자 모델(`User`)을 아래와 같이 생성한 후,

```bash
$ rails g devise User
      invoke  active_record
      create    db/migrate/20140528023115_devise_create_users.rb
      create    app/models/user.rb
      invoke    test_unit
      create      test/models/user_test.rb
      create      test/fixtures/users.yml
      insert    app/models/user.rb
       route  devise_for :users
```

이와 같은 작업으로 수행된 내용 중에 두가지 사항에 대해서 주목할 필요가 있다.

첫째, `User` 모델과 마이그레이션 클래스 파일이 생성된다.

데이터베이스 마이그레이션 작업을 하기 전에 한가지 고려해야 할 사항이 있다. 즉, 회원등록시 사용자가 등록한 본인의 이메일로 본인임을 확인하는 이메일을 발송할 것이기 때문에 방금 전에 생성된 `db/migrate/20140528023115_devise_create_users.rb` 파일을 열고 아래와 같이 `## Confirmable` 부분 아래의 네 줄과 하단의 세번째 `add_index` 부분의 커멘트 처리 부호를 제거하여 마이그레이션 작업시 실행되도록 한다. 그리고 `User` 모델 클래스에서 아래에 관련 strategy(`:confirmable`)를 추가한다.

```ruby
class User < ActiveRecord::Base
  # Include default devise modules. Others available are:
  # :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable,
         :confirmable
end
```

둘째, `config/routes.rb` 파일에 `devise`를 위한 라우트가 추가된다.

```ruby
Rails.application.routes.draw do
  devise_for :users
  root 'welcome#index'
end
```

아래와 같이 터미널의 커맨드라인 쉘에서 명령을 수행하면 `devise` 컨트롤러를 위한 라우팅 테이블을 확인(`$ rake routes`)할 수 있다

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/auth_blog/2014-05-28_11-59-07_zpsb791e7c5.png)

초록색으로 표시된 부분이 `devise` 컨트롤러에 대한 라우팅에 해당한다. 이 라우팅 테이블에 대한 자세한 설명은 생략하기로 한다. 자세한 내용을 원할 경우 [`여기`](http://guides.rubyonrails.org/routing.html#listing-existing-routes)를 참고하기 바란다.

변경한 마이그레이션 파일에 대해서 데이터베이스 마이그레이션을 수행한다.

```bash
$ rake db:migrate
== 20140528023115 DeviseCreateUsers: migrating ================================
-- create_table(:users)
   -> 0.0022s
-- add_index(:users, :email, {:unique=>true})
   -> 0.0010s
-- add_index(:users, :reset_password_token, {:unique=>true})
   -> 0.0005s
-- add_index(:users, :confirmation_token, {:unique=>true})
   -> 0.0005s
== 20140528023115 DeviseCreateUsers: migrated (0.0043s) =======================

마지막으로 `devise` 뷰 템플릿 파일들을 복사해서 입맛에 맞게 변경하기 쉽게 아래와 같이 실행한다.

```bash
$ rails g devise:views
      invoke  Devise::Generators::SharedViewsGenerator
      create    app/views/devise/shared
      create    app/views/devise/shared/_links.erb
      invoke  simple_form_for
      create    app/views/devise/confirmations
      create    app/views/devise/confirmations/new.html.erb
      create    app/views/devise/passwords
      create    app/views/devise/passwords/edit.html.erb
      create    app/views/devise/passwords/new.html.erb
      create    app/views/devise/registrations
      create    app/views/devise/registrations/edit.html.erb
      create    app/views/devise/registrations/new.html.erb
      create    app/views/devise/sessions
      create    app/views/devise/sessions/new.html.erb
      create    app/views/devise/unlocks
      create    app/views/devise/unlocks/new.html.erb
      invoke  erb
      create    app/views/devise/mailer
      create    app/views/devise/mailer/confirmation_instructions.html.erb
      create    app/views/devise/mailer/reset_password_instructions.html.erb
      create    app/views/devise/mailer/unlock_instructions.html.erb
```

실행 결과에서도 알 수 있듯이 `app/views/devise/` 디렉토리에 관련 뷰 템블릿 파일들이 생성된 것을 볼 수 있다. 위에서 다섯번째 줄을 보면 `form_for` 대신에 `simple_form_for` 모듈이 호출된 것을 볼 수 있는데, 이것은 `devise`에서 사용하는 폼 뷰 파일이 `simple_form`의 메소드를 사용하여 작성되도록 한 것이다.

지금까지 `devise`에 대한 셋업 과정이 문제없이 진행되었다면 어플리케이션내의 모든 컨트롤러와 뷰에서 사용할 수 있는 관련 헬퍼 메소드들이 제공된다.


## 브라우저에서의 확인

웹서버를 다시 구동하여 어플리케이션을 다시 실행한 후 브라우저에서 확인해 보자.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/auth_blog/2014-05-28_12-15-04_zpsf9c2c8a2.png)

실제로 이메일을 입력하고 비밀번호를 입력한 후 `Sign up` 버튼을 클릭하면 아래와 같은 이메일 화면이 또 다른 브라우저에 표시된다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/exploring_devise/2014-06-03_18-56-24_zps34a8ad23.png)

하단의 `Confirm my account` 링크를 클릭하면 회원등록 절차가  완료되고 아래와 같은 화면이 보이게 된다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/auth_blog/2014-05-28_12-29-10_zpsf3cdc62c.png)

위의 화면에서 1번이 표시된 부분을 보면 현재 루트로 이동한 것을 확인할 수 있고, 2번으로 표시된 메시지는 `flash` 메시지가 표시된 것이다. 이것은 디폴트 동작으로 이루어진 것이다.



이제 회원등록이 완료되고 로그인이 된 상태이다. 그러면 로그아웃을 해 보자. 이를 위해서 브라우저에서 아래와 같이 접속해 보자.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/auth_blog/2014-05-28_12-42-17_zps9ae22a88.png)

3번에서 보는 바와 같이 라우팅 테이블에는 `/users/sign_out`이 명시되어 있지만, 2번과 같이 `No route matches`와 같은 라우팅 에러가 발생한다. 브라우저에서 요청을 하게 되면 디폴트로 `GET` 메소드 요청이 되기 때문에 실제로 라우팅에서는 `delete` 메소드로 요청이 들어올 때의 `URI`만 인식하게 되는 것이다. 따라서 당연히 에러가 발생하게 되는 것이다.

이러한 문제는 뷰 템블릿에서 `link_to` 헬퍼 메소드를 사용하여 해결할 수 있다.

아래와 같이 `welcome` 컨트롤러의 `index` 액션 뷰 템플릿을 변경하자.

```html
<% if user_signed_in? %>
  <p>현재 로그인된 유저의 이메일 : <%= current_user.email %></p>
  <p><%= link_to "로그아웃", destroy_user_session_path, method: :delete, data: { confirm: "Are you sure?" } %></p>
<% else %>
  <p><%= link_to "로그인", new_user_session_path %></p>
<% end %>
```

이제, 로그인 상태에 따라 아래와 같이 화면에 표시될 것이다. 현재는 로그인 상태이므로 "현재 로그인된 유저의 이메일"(1번)을 표시하고 `로그아웃` 링크를 추가해 주었다. 브라우저상에서 `/users/sign_out`로 요청시 에러가 나던 것이 `link_to` 헬퍼메소드에 `:method` 옵션으로 `:delete`를 지정해 줌으로써 정확인 라우팅이 이루어지게 되었다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/auth_blog/2014-05-28_13-43-42_zps57e6a987.png)

다음으로, 2번 `로그아웃` 링크를 클릭하여 세션을 해제하도록 하자.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/auth_blog/2014-05-28_13-50-25_zpsbfd69ef1.png)

확인 창이 나타나고,

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/auth_blog/2014-05-28_13-51-26_zps3ba92fe9.png)

로그아웃 상태로 돌아간다.

지금까지 회원가입 및 인증에 대해서 작업을 했다.
다음은 `bootstrap`에 맞게 어플리케이션 레이아웃을 변경하고 `flash` 메시지도 제대로 보이도록 할 것이다.

---

_**References:**_

1. [Listing Existing Routes](http://guides.rubyonrails.org/routing.html#listing-existing-routes)
