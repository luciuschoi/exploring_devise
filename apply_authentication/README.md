# 사용자 인증 적용하기

`devise`는 모든 컨트롤러와 액션에서 사용할 수 있는 헬퍼메소드를 제공해 준다.

* `before_action :authenticate_user!`
* `current_user`
* `user_signed_in?`
* `user_session`

## Post 리소스의 생성

인증 효과를 검증하기 위해서 여기서 `Post` 리소스를 생성하기로 하자.

```bash
$ rails g scaffold Post title content:text user:references
      invoke  active_record
      create    db/migrate/20140528081107_create_posts.rb
      create    app/models/post.rb
      invoke    test_unit
      create      test/models/post_test.rb
      create      test/fixtures/posts.yml
      invoke  resource_route
       route    resources :posts
      invoke  scaffold_controller
      create    app/controllers/posts_controller.rb
      invoke    erb
      create      app/views/posts
      create      app/views/posts/index.html.erb
      create      app/views/posts/edit.html.erb
      create      app/views/posts/show.html.erb
      create      app/views/posts/new.html.erb
      create      app/views/posts/_form.html.erb
      invoke    test_unit
      create      test/controllers/posts_controller_test.rb
      invoke    helper
      create      app/helpers/posts_helper.rb
      invoke      test_unit
      create        test/helpers/posts_helper_test.rb
      invoke    jbuilder
      create      app/views/posts/index.json.jbuilder
      create      app/views/posts/show.json.jbuilder
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/posts.js.coffee
      invoke    scss
      create      app/assets/stylesheets/posts.css.scss
      invoke  scss
      create    app/assets/stylesheets/scaffolds.css.scss
```

`Post` 모델의 속성으로 지정한 `user:references`는 두가지 일을 한다. 하나는 foreign key로 `user_id:integer`를 만들고 인덱스도 동시에 생성한다.

```ruby
class CreatePosts < ActiveRecord::Migration
  def change
    create_table :posts do |t|
      t.string :title
      t.text :content
      t.references :user, index: true

      t.timestamps
    end
  end
end
```

다른 하나는 `Post` 모델 클래스에 `belongs_to :user`를 자동으로 추가해 준다.

```ruby
class Post < ActiveRecord::Base
  belongs_to :user
end
```

물론 `user_id:integer:index`와 같이 지정해도 되지만, 이 때는 `Post` 모델 클래스에 `belongs_to :user`를 추가해 주지 않아 직접 작성해 주어야 한다.

```ruby
class CreatePosts < ActiveRecord::Migration
  def change
    create_table :posts do |t|
      t.string :title
      t.text :content
      t.integer :user_id

      t.timestamps
    end
    add_index :posts, :user_id
  end
end
```

그리고 데이터베이스 마이그레이션을 한다.

```bash
$ rake db:migrate
== 20140528081107 CreatePosts: migrating ======================================
-- create_table(:posts)
   -> 0.0048s
== 20140528081107 CreatePosts: migrated (0.0049s) =============================
```

`User` 모델 클래스(app/models/user.rb)에는 아래와 같이 관계 선언을 추가해 주어 `Post` 모델(app/models/post.rb)에 대해서 일대다로 연결되도록 한다.

```ruby
class User < ActiveRecord::Base
  has_many :posts, dependent: :destroy
end
```

`has_many` 메소드에 사용한 `dependent: :destroy` 옵션은 특정 `user` 객체를 삭제할 때 연관된 모든 `post` 객체들을 한꺼번에 삭제하도록 해 주어야 하는데, 이는 `user_id` 값이 없는 `post` 레코드들(`'orphan', 부모 없는 고아` )은 의미 없는 쓰레기 데이터가 될 수 있기 때문이다.

## Post 리소스에 대한 접근제한하기

우선, 로그인 하지 않은 상태에서는 `posts` 컨트롤러의 `index`, `show` 액션에 대한만 접근은 할 수 있도록 하고, 로그인 후에 비로소 `new`, `edit` 액션에도 접근할 수 있도록 하는 권한 설정을 만들어 보자.

이를 위해서 `app/controllers/posts_controller.rb` 컨트롤러 클래스 파일을 열어서 상단에 `before_action :authenticate_user!`를 추가해 준다. 그리고 `:only` 옵션을 이용해서 인증이 필요한 액션들의 심볼명을 배열로 만들어 준다.

```ruby
before_action :authenticate_user!, only: [ :new, :edit, :create, :update, :destroy ]
```

따라서 `index`, `show` 액션은 인증이 필요없게 되는 것이다.

그러나, 다시 한번 생각해 보면 모든 액션에 대해서 디폴트로 로그인한 상태를 강요하고 혹시 추가되는 액션에 대해서도 우선은 로그인을 해야하는 디폴트 상태를 가지도록 하는 것이 맞는 것 같다. 만약 특정 액션은 로그인하지 않은 상태에서도 접근할 수 있도록 하기 위해서는 `:except` 옵션에 해당 액션을 추가하는 로직이 제대로인 것 같다. 따라서 위의 `:only` 대신에 블랙리스트 방식의 `:except` 옵션을 지정하도록 한다.

```ruby
before_action :authenticate_user!, except: [ :index, :show ]
```

이제 브라우저에서 로그인하지 않는 상태에서 `Posts` 메뉴를 클릭한다.

![](/assets/2014-05-28_19-33-59_zpsd9d2a758.png)

이 때 `Posts` 페이지에서 `News post`(1번) 링크를 클릭하면 아래와 같이 로그인 페이지로 이동한 후, 인증 실패에 대한 `flash` 메시지(1번)를 상단에 보여 주게 된다.

![](/assets/2014-05-28_19-35-11_zpsbbc758f8.png)

과연 우리가 의도했던 데로 처리되었다. 로그인 후에 이와 같은 동작을 반복하면 이 때는 인증 실패에 대한 메시지가 나타나지 않고 바로 글을 입력할 수 있는 폼 페이지로 이동하게 된다.

![](/assets/2014-05-28_19-42-22_zpsfbc0aa40.png)

로그인 후에는 1번 위치에 로그인된 사용자의 이메일 주소가 나타난다. 데이터를 입력하고 `Create Post` 링크 버튼(2번)을 클릭하면 데이터가 저장된 후 `show` 액션으로 리디렉트된다.

![](/assets/2014-05-28_19-45-45_zps26e55199.png)

그러나 `flash` 메시지가 중복(1번)되어 나타나고 2번의 `User` 항목에는 알 수 없는 숫자들이 보인다. 이것은 `user` 객체의 `id` 값을 나타내는데 사용자들에게는 도움이 되지 못하는 정보이며 실제 사용자의 이메일 주소를 보이도록 하면 도움이 될 것 같다.

이 두가지 사항은 아래와 같이 수정한다.

```html
<p>
  <strong>Title:</strong>
  <%= @post.title %>
</p>

<p>
  <strong>Content:</strong>
  <%= @post.content %>
</p>

<p>
  <strong>User:</strong>
  <%= @post.user.present? ? @post.user.email : "n/a" %>
</p>

<%= link_to 'Edit', edit_post_path(@post), class: 'btn btn-default' %>
<%= link_to 'Back', posts_path, class: 'btn btn-default' %>
```

다시 해당 글을 수정한 후 저장하면 아래와 같이 깔끔하게 보이게 된다.

![](/assets/2014-05-28_20-02-26_zps577b0438.png)

