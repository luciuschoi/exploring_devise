# Bootstrap 클래스로 Styling 업데이트하기

이제부터는 `Bootstrap`에서 제공하는 다양한 클래스를 이용하여 기존의 뷰 템플릿을 업데이트해 보자.

우선 `app/views/posts/_form.html.erb`을 아래와 같이 변경한다.

```html
<%= simple_form_for(@post) do |f| %>
  <%= f.error_notification %>

  <div class="form-group">
    <%= f.input :title, input_html: { class: 'form-control' } %>
  </div>
  <div class="form-group">
    <%= f.input :content, input_html: { class: 'form-control' } %>
  </div>
  <div class="form-group">
    <%= f.association :user, input_html: { class: 'form-control' } %>
  </div>
  <%= f.button :submit, class: 'btn btn-default'%>

<% end %>
```

`simple_form` 젬은 레일스의 폼 헬퍼 메소드를 간단하게 사용할 수 있도록 도와 준다. 위의 소스코드와 같이 레일스의 폼 헬퍼에서는 보지 못하는 폼 핸들러의 `input` 메소드는 속성명을 심볼로 받아 표준 html 폼 태그로 변환해 준다. 자세한 내용은 `simple_form` [`문서`](https://github.com/plataformatec/simple_form)를 참고하기 바란다.

참고로, `<%= f.error_notification %>`는 폼을 서밋했을 때 해당 모델의 유효성 검증 절차가 진행되는데 이 때 검증오류를 표시해 준다.
![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/auth_blog/2014-05-29_08-37-03_zpsa5190464.png)

1번은 작성자를 선택하는 폼의 `select` 엘리먼트인데, 사실 이것은 입력 폼에 보여줄 필요는 없는 것이다. 현재 로그인한 사용자의 정보를 알 수 있게 때문에 `create` 액션에서 `current_user`를 이용하여 값을 할당해 주면 된다. 따라서 이 `select` 엘리먼트는 삭제하고 `posts_controller.rb` 파일을 열어 `create` 액선을 아래와 같이 수정한다. 2번 링크는 `app/views/posts/edit.html.erb` 파일에서 수정할 예정이다.

```ruby
def create
  @post = Post.new(post_params)
  @post.user = current_user

  respond_to do |format|
    if @post.save
      format.html { redirect_to @post, notice: 'Post was successfully created.' }
      format.json { render :show, status: :created, location: @post }
    else
      format.html { render :new }
      format.json { render json: @post.errors, status: :unprocessable_entity }
    end
  end
end
```

기존 코드에서 세번째 줄에 있는 `@post.user = current_user` 만을 추가했다. 이후에 `@post.save` 명령이 실행될 때 이 객체의 `user_id` 속성에 현재 로그인한 사용자의 `id`가 저장된다.

이제 `app/views/posts/new.html.erb`을 열고 아래와 같이 수정한다.

```html
<h2>New post</h2>
<div class="row">
  <div class='col-md-12'>
    <p style='text-align:right;'>
      created by <%= current_user.email %>, Current Time : <%= Time.now %>
    </p>
  </div>
</div>
<%= render 'form' %>

<hr>
<%= link_to 'Back', posts_path, class: "btn btn-default" %>
```

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/auth_blog/2014-05-29_09-10-25_zps8e3c1181.png)

1번 새글을 작성하는 현재 로그인 사용자의 이메일 주소를 보이게 하고 2번에서는 현재 시각을 표시하도록 했다.

다음 `app/views/posts/edit.html.erb` 파일을 열고 아래와 같이 수정한다.

```html
<h2>Editing post </h2>
<div class="row">
  <div class='col-md-12'>
    <p style='text-align:right;'>
      created by <%= current_user.email %>, <%= time_ago_in_words(@post.created_at) %> ago
    </p>
  </div>
</div>
<%= render 'form' %>

<hr>
<%= link_to 'Show', @post, class: 'btn btn-default' %>
<%= link_to 'Back', posts_path, class: 'btn btn-default' %>
```

[`time_ago_in_words`](http://api.rubyonrails.org/classes/ActionView/Helpers/DateHelper.html#method-i-time_ago_in_words) 헬퍼 메소드는 `Datetime` 시간을 상대시간으로 계산하여 표시해 준다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/auth_blog/2014-05-29_09-22-20_zpsbd63393d.png)

1번은 글을 작성한 사용자의 이메일 주소를 보이게 하고 2번에서는 최초 작성된 시간을 현재 시간 기준으로 표시해 준다.

이제는 `app/views/posts/index.html.erb` 파일을 열어서 아래와 같이 수정 한다.

```html
<h2>Listing posts</h2>

<table class="table">
  <thead>
    <tr>
      <th>Title</th>
      <th>Created by</th>
      <th>Actions</th>
    </tr>
  </thead>

  <tbody>
    <% @posts.each do |post| %>
      <tr>
        <td class="col-md-6 col-xs-6"><%= post.title %></td>
        <td><%= post.user.present? ? post.user.email : "n/a" %>, <%= time_ago_in_words(post.created_at) %> ago</td>
        <td>
          <%= link_to "<span class='glyphicon glyphicon-eye-open'></span>".html_safe, post %>&nbsp;
          <%= link_to "<span class='glyphicon glyphicon-edit'></span>".html_safe, edit_post_path(post) %>&nbsp;
          <%= link_to "<span class='glyphicon glyphicon-trash'></span>".html_safe, post, method: :delete, data: { confirm: 'Are you sure?' } %>
        </td>
      </tr>
    <% end %>
  </tbody>
</table>

<br>

<%= link_to 'New Post', new_post_path, class: "btn btn-default" %>
```

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/auth_blog/2014-05-29_09-48-13_zpsedb00786.png)


1번은 글을 작성한 사용자의 id 값(`@post.user_id`)이 없을 때 `n/a`로 표시하도록 했다. 여기서는 루비의 `ternary` 연산자([condition] `?` [true expression] `:` [false expression])를 사용하였다. 2번은 문자열로 보이던 링크 표시를 아이콘으로 표시하도록 했다. 이것에 대한 자세한 설명은 [`여기`](http://getbootstrap.com/components/#glyphicons-how-to-use)를 참고하기 바랍니다.

`show` 액션에 대한 뷰 템플릿 파일(`app/views/posts/show.html.erb`)을 열어 아래와 같이 수정한다.
```html
<div class='post'>
  <div class="title">
    <h3>Title:
    <%= @post.title %></h3>
  </div>

  <div class="content">
    <%= simple_format @post.content %>
  </div>

  <div class="user">
    <strong>Created by </strong>
    <%= @post.user.present? ? @post.user.email : "an anonymous user" %>,
    <%= time_ago_in_words(@post.created_at) %> ago
  </div>
</div>

<hr>
<%= link_to 'Edit', edit_post_path(@post), class: 'btn btn-default' %>
<%= link_to 'Back', posts_path, class: 'btn btn-default' %>
```

그리고 `app/assets/stylesheets/application.css.scss` 파일을 열어 아래에 `@import 'posts';`을 추가한다.

```css
$light-sky: #d8f7ff;
$navbar-default-color: $light-sky;
$navbar-default-bg: #293370;
$navbar-default-link-color: #c1fff9;
$navbar-default-link-active-color: $light-sky;
$navbar-default-link-hover-color: white;
$navbar-default-link-hover-bg: black;

@import "bootstrap-custom";

body { margin-top: 60px; }

@import "posts";
```

다음은 `app/assets/stylesheets/posts.css.scss` 파일을 열어 아래와 같이 추가해 준다.

```css
.post {
  border:1px solid #eaeaea;
  padding:1em;
  .title {
    border-bottom: 1px solid #eaeaea;
    margin-bottom:1em;
  }
  .content {

  }
  .user {
    border-top: 1px solid #eaeaea;
    margin-top: 1em;
    text-align: right;
  }
}
```

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/auth_blog/2014-05-29_10-19-16_zps9cd435cb.png)

마지막으로 `index` 뷰에서 글 목록의 나열 순서를 최신글이 위로 오도록 해보자. `app/controllers/post_controller.rb` 파일을 열어 `index` 액션을 아래와 같이 수정하자.

```ruby
def index
  @posts = Post.order(created_at: :desc)
end
```

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/auth_blog/2014-05-29_10-34-23_zpse1069426.png)


## Devise 뷰


### > Sign Up

`app/views/devise/registrations/new.html.erb` 파일을 열고 아래와 같이 수정한다.

```html
<h2>Sign up</h2>

<%= simple_form_for(resource, as: resource_name, url: registration_path(resource_name)) do |f| %>
  <%= f.error_notification %>

  <div class="form-group">
    <%= f.input :email, required: true, autofocus: true, input_html: { class: 'form-control' } %>
  </div>
  <div class="form-group">
    <%= f.input :password, required: true, input_html: { class: 'form-control' } %>
  </div>
  <div class="form-group">
    <%= f.input :password_confirmation, required: true, input_html: { class: 'form-control' } %>
  </div>
  <%= f.button :submit, "Sign up", class: "btn btn-default" %>
<% end %>

<%= render "devise/shared/links" %>
```

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/auth_blog/2014-05-29_11-46-03_zpsc7cf258e.png)




### > Sign In

`app/views/devise/sessions/new.html.erb` 파일을 열고 아래와 같이 수정한다.

```html
<h2>Sign in</h2>

<%= simple_form_for(resource, as: resource_name, url: session_path(resource_name)) do |f| %>
  <div class="form-group">
    <%= f.input :email, required: false, autofocus: true, input_html: { class: 'form-control' } %>
  </div>
  <div class="form-group">
    <%= f.input :password, required: false, input_html: { class: 'form-control' } %>
  </div>
  <div class="form-group col-md-12">
    <%= f.input :remember_me, as: :boolean, inline_label: true, label: false if devise_mapping.rememberable? %>
  </div>
  <%= f.button :submit, "Sign in", class: 'btn btn-default' %>
<% end %>

<%= render "devise/shared/links" %>
```

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/auth_blog/2014-05-29_11-49-18_zps7546b970.png)

### Edit Profile

`app/views/devise/registrations/edit.html.erb` 파일을 열고 아래와 같이 수정한다.

```html
<h2>Edit <%= resource_name.to_s.humanize %></h2>

<%= simple_form_for(resource, as: resource_name, url: registration_path(resource_name), html: { method: :put }) do |f| %>
  <%= f.error_notification %>

  <div class="form-group">
    <%= f.input :email, required: true, autofocus: true, input_html: { class: 'form-control' } %>
  </div>
  <div class="form-group">
    <% if devise_mapping.confirmable? && resource.pending_reconfirmation? %>
      <p>Currently waiting confirmation for: <%= resource.unconfirmed_email %></p>
    <% end %>
  </div>

  <div class="form-group">
    <%= f.input :password, autocomplete: "off", hint: "leave it blank if you don't want to change it", required: false, input_html: { class: 'form-control' } %>
  </div>
  <div class="form-group">
    <%= f.input :password_confirmation, required: false, input_html: { class: 'form-control' } %>
  </div>
  <div class="form-group">
    <%= f.input :current_password, hint: "we need your current password to confirm your changes", required: true, input_html: { class: 'form-control' } %>
  </div>
  <%= f.button :submit, "Update", class: 'btn btn-default' %>
<% end %>

<hr>

<h3>Cancel my account</h3>

<p>Unhappy? <%= link_to "Cancel my account", registration_path(resource_name), data: { confirm: "Are you sure?" }, method: :delete %></p>

<%= link_to "Back", :back, class: "btn btn-default" %>
```

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/auth_blog/2014-05-29_11-54-42_zps21a5a5a2.png)

### > Welcome Page (app/views/welcome/index.html.erb)

```html
<center>
<h1>Welcome to</h1>

<div style="margin-bottom:2em;">
    <%= image_tag "auth_blog_emblem_300.png", size: "200", style: "margin:2em 0"%>
</div>

<% if user_signed_in? %>
  <p><%= link_to "로그아웃", destroy_user_session_path, method: :delete, data: {confirm: "Are you sure?" }, class: 'btn btn-default'%></p>
<% else %>
  <p><%= link_to "로그인", new_user_session_path, class: 'btn btn-default' %></p>
<% end %>
</center>
```

여기서 사용한 `AuthBlog` 엠블렘 이미지(`auth_blog_emblem_300.png`)는 [`여기`](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/exploring_devise/auth_blog_emblem_300_zps8623092b.png)를 클릭하면 다운로드 받을 수 있다.

![](http://i1373.photobucket.com/albums/ag392/rorlab/Photobucket%20Desktop%20-%20RORLAB/auth_blog/2014-05-29_12-11-28_zpsdacf4564.png)
