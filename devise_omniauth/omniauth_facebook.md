# Facebook으로 로그인하기

우선 `Facebook`에 연결해서 로그인할 수 있도록 구현해 보자.
이를 위해서 필요한 젬을 `Gemfile`에 추가하고 인스톨한다.

```ruby
gem 'omniauth-facebook'
```

`User` 모델에 `name`, `provider`, `uid`, `image` 속성을 추가하고 아래와 같이 DB 마이그레이션한다.

* `name` : 페이스북 사용자명을 저장할 속성
* `provider` : 인증서비스 제공업체명을 저장할 속성 (예, facebook, twitter 등)
* `uid` : 사용자 아이디를 저장할 속성
* `image` : 페이스북의 프로필 이미지 주소를 저장할 속성

```bash
$ rails g migration add_columns_to_users name provider uid image
      invoke  active_record
      create    db/migrate/20140601051431_add_columns_to_users.rb
$ rake db:migrate
== 20140601051431 AddColumnsToUsers: migrating ================================
-- add_column(:users, :name, :string)
   -> 0.0025s
-- add_column(:users, :provider, :string)
   -> 0.0002s
-- add_column(:users, :uid, :string)
   -> 0.0002s
-- add_column(:users, :image, :string)
   -> 0.0002s
== 20140601051431 AddColumnsToUsers: migrated (0.0033s) =======================
```

다음은 `config/initializers/devise.rb` 파일을 열고 아래의 내용을 추가한다.

```ruby
config.omniauth :facebook, "APP_ID", "APP_SECRET"
```

이 때 필요한 `APP_ID`와 `APP_SECRET` 값을 얻기 위해서
https://developers.facebook.com 로 접속하여 상단 메뉴 중 좌측에 잇는 `App` 항목을 클릭하면 새로운 어플리케이션을 어렵지 않게 생성할 수 있다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/exploring_devise/2014-06-01_14-28-13_zps8f430dc6.png)

1번과 2번 값을 직접 사용하면 보안상의 문제가 될 수 있다. 이 문제를 간단하게 해결하기 위해서 로컬 터미널 운영 시스템의 환경변수를 이용하도록 하자.

```ruby
config.omniauth :facebook, ENV["FB_APP_ID"], ENV["FB_APP_SECRET"]
```

로컬 터미널 커맨드라인 쉘에서 아래와 같이 명령을 실행한다.

```bash
$ echo "export FB_APP_ID=XXXXXXXXXXXXXXX" >> ~/.bashrc
$ echo "export FB_APP_SECRET=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX" >> ~/.bashrc
$ source ~/.bashrc
```

> **Caution** 환경변수를 사용하였기 때문에 허로쿠로 배포한 후에는 아래와 같이 허로쿠로 이 환경변수들을 지정해 주어야 합니다.

```bash
$ heroku config:set FB_APP_ID=XXXXXXXXXXXXXXX
$ heroku config:set FB_APP_SECRET=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```


> **Info** zsh 쉘을 사용할 경우에는 `~/.bashrc` 대신에 `~/.zshrc` 파일명을 사용하면 된다.

방금 추가한 환경변수가 제대로 셋팅되었는지 아래와 같이 확인할 있다.

```bash
$ env | grep FB
FB_APP_ID=XXXXXXXXXXXXXXX
FB_APP_SECRET=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```
한가지 추가로 설정해 주어야 할 것은 아래 그림과 같이 App > Settings > Advanced 탭으로 이동하여 4번 위치의 `Valid OAuth redirect URIs` 값을 우선 `http://localhost:3000`로 입력하고 5번에서 `변경내용저장`을 클릭하여 작업을 완료한다. 이것은 `Facebook`을 호출한 후 콜백할 웹어플리케이션의 주소를 입력해 주는 것이다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/exploring_devise/2014-06-01_16-23-38_zpsf8d38f5f.png)

> **Note** 허로쿠로 배포할 경우에는 4번에 허로쿠 주소도 함께 등록해 주어야 한다.

이제 `User` 모델 클래스 파일을 열고 아래의 내용을 추가해 준다.

```ruby
devise :omniauthable, :omniauth_providers => [:facebook]
```

결국 아래와 같이 된다.

```ruby
devise :database_authenticatable, :registerable,
       :recoverable, :rememberable, :trackable, :validatable,
       :omniauthable, :omniauth_providers => [:facebook]
```

이와 같이 추가해 주면 아래와 같은 `omniauth`용 라우팅이 자동으로 생성된다. 터미널 쉘에서 아래와 같이 확인할 수 있다.

```bash
$ rake routes CONTROLLER=devise/omniauth_callbacks
                 Prefix Verb     URI Pattern                            Controller#Action
user_omniauth_authorize GET|POST /users/auth/:provider(.:format)        devise/omniauth_callbacks#passthru {:provider=>/facebook/}
 user_omniauth_callback GET|POST /users/auth/:action/callback(.:format) devise/omniauth_callbacks#(?-mix:facebook)
```

위의 내용을 보기 좋게 정리하면 아래와 같다

|Items|Values|
|---|---|
|Prefix|(1) _**user_omniauth_authorize**_|
|Verb|GET/POST|
|URI Pattern|/users/auth/:provider(.:format)|
|Controller#Action|devise/omniauth_callbacks#passthru {:provider=>/facebook/}|
|**Items**|**Values**|
|Prefix|(2) _**user_omniauth_callback**_|
|Verb|GET/POST|
|URI Pattern|/users/auth/:action/callback(.:format)|
|Controller#Action|devise/omniauth_callbacks#(?-mix:facebook)|

이 두 개의 `omniauth`용 라우팅 `Prefix` 중에 실제 (2)를 직접 사용할 일은 거의 없다.

실제로 `OmniauthCallbacks` 컨트롤러(`omniauth_callbacks_controller.rb`)를 `app/controllers/users/` 디렉토리에 생성하고 아래와 같이 작성한다.

```ruby
class Users::OmniauthCallbacksController < Devise::OmniauthCallbacksController
  def facebook
    # You need to implement the method below in your model (e.g. app/models/user.rb)
    @user = User.find_for_facebook_oauth(request.env["omniauth.auth"])

    if @user.persisted?
      sign_in_and_redirect @user, :event => :authentication #this will throw if @user is not activated
      set_flash_message(:notice, :success, :kind => "Facebook") if is_navigational_format?
    else
      session["devise.facebook_data"] = request.env["omniauth.auth"]
      redirect_to new_user_registration_url
    end
  end
end
```

이것은 `omniauth-facebook` 젬 문서에 나와 있는 것이므로 일단 복사해서 추가한다.

> **Note** 클래스명 앞에 `Users::` 접두사가 붙는 것에 유의하기 바란다. 컨트롤러의 네임스페이스를 이와 같이 추가하는데 대개는 동일한 이름의 디렉토리에 위치시킨다.

그리고 `devise_for :users`를 아래와 같이 수정하여 실제로 해당 콜백 컨트롤러의 위치를 지정해 준다.

```ruby
devise_for :users, :controllers => { :omniauth_callbacks => "users/omniauth_callbacks" }
```

다음은 `User` 모델 클래스 파일(`app/models/user.rb`)에 아래와 같이 두개의 메소드를 추가한다.

[첫번째 메소드]

```ruby
def self.find_for_facebook_oauth(auth)
  user = where(auth.slice(:provider, :uid)).first_or_create do |user|
    user.provider = auth.provider
    user.uid = auth.uid
    user.email = auth.info.email
    user.password = Devise.friendly_token[0,20]
    user.name = auth.info.name   # assuming the user model has a name
    user.image = auth.info.image # assuming the user model has an image
  end

  # 이 때는 이상하게도 after_create 콜백이 호출되지 않아서 아래와 같은 조치를 했다.
  user.add_role :user if user.roles.empty?
  user   # 최종 반환값은 user 객체이어야 한다.
end
```
[두번째 메소드]

```ruby
def self.new_with_session(params, session)
  super.tap do |user|
    if data = session["devise.facebook_data"] && session["devise.facebook_data"]["extra"]["raw_info"]
      user.email = data["email"] if user.email.blank?
    end
  end
end
```

이제 (1) 경로 헬퍼메소드를 이용하여 링크를 메뉴항목으로 추가해 보자.

어플리케이션 레이아웃 파일(`app/views/layouts/application.html.erb`)을 열고 아래와 같이 보이도록 `user_omniauth_authorize_path(:facebook)` 헬퍼 메소드를 추가한다.

```html
<ul class="nav navbar-nav navbar-right">
  <% if user_signed_in? %>
    <li class="dropdown">
      <a href="#" class="dropdown-toggle" data-toggle="dropdown">
        <%= image_tag(current_user.image, size: '20x20', style:'border-radius:3px;vertical-align:top;') if current_user.image %>
        <%= current_user.name %>
        <b class="caret"></b>
      </a>
      <ul class="dropdown-menu">
        <li><%= link_to "My Profile", edit_user_registration_path %></li>
        <li><%= link_to "Roles : " + user_roles(current_user), '#' %></li>
      </ul>
    </li>
    <li><%= link_to "Sign out", destroy_user_session_path, method: :delete, data: { confirm: "Are you sure?" } %></li>
  <% else %>
    <li class="dropdown">
      <a href="#" class="dropdown-toggle" data-toggle="dropdown">Sign in
       <b class="caret"></b></a>
      <ul class="dropdown-menu">
        <li><%= link_to "with Auth Blog", new_user_session_path %></li>
        <li><%= link_to "with Facebook", user_omniauth_authorize_path(:facebook) %></li>
      </ul>
    </li>
    <li><%= link_to "Sign up", new_user_registration_path %></li>
  <% end %>
</ul>
```

그리고 `welcome#index` 뷰 템플릿 파일(`app/views/welcome/index.html.erb`)을 열고 아래와 같이 변경한다.

```html
<% if user_signed_in? %>
  <p><%= link_to "로그아웃", destroy_user_session_path, method: :delete, data: {confirm: "Are you sure?" }, class: 'btn btn-default'%></p>
<% else %>
  <p><%= link_to "로그인", new_user_session_path, class: 'btn btn-default' %></p>
  <p><%= link_to "페이스북으로 로그인", user_omniauth_authorize_path(:facebook) %></p>
<% end %>
```

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/exploring_devise/2014-06-01_23-38-42_zpsdcbcb2e5.png)

이제 `로그인` 버튼 아래에 있는 `페이스북으로 로그인` 링크나 우측 상단의 `Sign in` 메뉴의 `with Facebook` 항목을 클릭하면 아래와 같은 화면이 보이고,

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/exploring_devise/2014-06-01_17-12-44_zpsdfc86392.png)

이 웹어플리케이션에서 본인의 페이스북 계정으로 최초로 로그인할 때 단 한번만 아래와 같은 확인 메시지를 볼 수 있다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/exploring_devise/2014-06-01_16-20-36_zpsa258622f.png)

로그인이 완료된 후의 화면은 아래와 같다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/exploring_devise/2014-06-02_01-35-56_zpsa31d0904.png)

1번은 로그인한 사용자의 페이스북 사용자명, 2번은 페이스북으로 인증이 성공적으로 처리됨을 알리는 `flash` 메시지, 3번은 로그아웃을 할 수 있는 링크를 말한다.

이제 페이스북으로 로그인해 보자.

지금까지 `omniauth-facebook` 젬을 이용하여 로그인하는 방법을 알아 보았다.

---

_**References:**_

1. [rails 4 omniauth using devise with twitter, facebook and linkedin](http://sourcey.com/rails-4-omniauth-using-devise-with-twitter-facebook-and-linkedin/)
2. [Linux: List All Environment Variables Command
](http://www.cyberciti.biz/faq/linux-list-all-environment-variables-env-command/)
3. [Heroku : Configuration and Config Vars](https://devcenter.heroku.com/articles/config-vars)
