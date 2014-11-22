# 어플리케이션 레이아웃 작업

우선 모바일 디바이스에서도 웹페이지의 scale이 제대로 보이게 하기 위해서는 아래와 같이 `<head></head>` 사이에 아래와 같이 메타 정보를 추가해 주어야 한다.

```html
<meta charset="utf-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0" />
```

지금까지는 컨텐츠들이 웹페이지에 제대로 보이지 않았다. 이제는 `Bootstrap`의 `CSS` 클래스를 적용해서 어플리케이션 레이아웃을 아래와 같이 변경해 보자.

```html
<body>
    <!-- Fixed navbar -->
    <div class="navbar navbar-default navbar-fixed-top" role="navigation">
      <div class="container">
        <div class="navbar-header">
          <button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse">
            <span class="sr-only">Toggle navigation</span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
          </button>
          <a class="navbar-brand" href="#">AuthBlog</a>
        </div>
        <div class="navbar-collapse collapse">
          <ul class="nav navbar-nav">
            <li class="active"><a href="#">Home</a></li>
            <li><a href="#about">About</a></li>
            <li><a href="#contact">Contact</a></li>
            <li class="dropdown">
              <a href="#" class="dropdown-toggle" data-toggle="dropdown">Dropdown <b class="caret"></b></a>
              <ul class="dropdown-menu">
                <li><a href="#">Action</a></li>
                <li><a href="#">Another action</a></li>
                <li><a href="#">Something else here</a></li>
                <li class="divider"></li>
                <li class="dropdown-header">Nav header</li>
                <li><a href="#">Separated link</a></li>
                <li><a href="#">One more separated link</a></li>
              </ul>
            </li>
          </ul>
        </div><!--/.nav-collapse -->
      </div>
    </div>

    <div class="container">

      <%= yield %>

    </div> <!-- /container -->
</body>
```

이제, 브라우저 화면을 다시 로드하면 아래와 같이 보이게 된다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/auth_blog/2014-05-28_15-37-03_zps2001566c.png)

## flash 메시지 표시하기

지금까지는 `flash` 메시지가 제대로 보이지 않았다. 세가지 작업을 추가하면 아래와 같이 이쁘게 보이게 할 수 있다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/auth_blog/2014-05-28_15-42-16_zps52949be1.png)

`app/helpers/application_helper.rb` >

```ruby
module ApplicationHelper
  def bootstrap_class_for(flash_type)
    case flash_type
      when "success"
        "alert-success"   # Green
      when "error"
        "alert-danger"    # Red
      when "alert"
        "alert-warning"   # Yellow
      when "notice"
        "alert-info"      # Blue
      else
        flash_type.to_s
    end
  end
end
```

`app/views/layouts/application.html.erb` >

```html
<div class="container">
  <%= render partial: "shared/flash_messages", flash: flash %>
  <%= yield %>
</div>
```

`app/views/shared/_flash_messages.html.erb` >

```html
<% flash.each do |type, message| %>
  <div class="alert <%= bootstrap_class_for(type) %> alert-dismissable fade in">
    <button type="button" class="close" data-dismiss="alert" aria-hidden="true">&times;</button>
    <%= message %>
  </div>
<% end %>
```

이제 `로그아웃` 링크를 클릭하면와 같이 `flash` 메시지가 보이게 된다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/auth_blog/2014-05-28_16-06-20_zpsa1b4765d.png)

다음은 사용자의 로그인 상태를 상단 메뉴바로 옮겨 보자.

```html
<div class="navbar-collapse collapse">
  <ul class="nav navbar-nav">
    <li class="active"><%= link_to "Home", root_path %></li>
    <li><%= link_to "Posts", "#" %></li>
  </ul>
  <ul class="nav navbar-nav navbar-right">
    <% if user_signed_in? %>
      <li class="dropdown">
        <a href="#" class="dropdown-toggle" data-toggle="dropdown"><%= current_user.email %> <b class="caret"></b></a>
        <ul class="dropdown-menu">
          <li><%= link_to "My Profile", edit_user_registration_path %></li>
          <li><%= link_to "Sign out", destroy_user_session_path, method: :delete, data: { confirm: "Are you sure?" } %></li>
        </ul>
      </li>
    <% else %>
      <li><%= link_to "Sign in", new_user_session_path %></li>
      <li><%= link_to "Sign up", new_user_registration_path %></li>
    <% end %>
  </ul>
</div>
```

브라우저에서 변경내용을 확인하면 아래와 같다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/auth_blog/2014-05-28_19-20-40_zpsed335edb.png)

`Home`(1번)을 클릭하면 루트로 이동하고  `Posts`(2번)을 클릭하면 게시물을 작성하는 페이지로 이동한다(다음 섹션에서 작성할 예정이며 여기서는 링크만 만들어 두기로 한다). 그리고 오른쪽에는 `Sign in`(3번)과 `Sign out`(4번) 메뉴를 두고 클릭하면 각각 `로그인`, `회원가입`하는 페이지로 이동하도록 했다.

로그인하면 아래와 같이 로그인 메뉴항목(2번)이 변경된다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/auth_blog/2014-05-28_19-26-55_zps1303c20a.png)

이제 메뉴바가 어느 정도 정리가 되었으니 다음 섹션으로 넘어 가도록 하자.
