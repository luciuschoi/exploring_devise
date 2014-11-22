# 업데이트 로그 (Change Logs)

### v1.0.3 : 2014.8.30

* Introduction 챕터 오타 수정, [Dev_Senna](mailto:dev.senna@gmail.com )

```
쉽지 많은 => 쉽지만은
```


### v1.0.2 : 2014.6.7

* 어플리케이션 레이아웃 파일의 `<head></head>` 태그 사에 아래의 내용을 추가하여 모바일 디바이스에서도 제대로 보이게 함.

  ```html
  <meta charset="utf-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width,   initial-scale=1.0, maximum-scale=1.0, user-scalable=0" />
```


* `welcome#index` 뷰 템플릿 파일에서 엠블렘 이미지의 크기와 margin 값을 변경함.

  ```html
  <%= image_tag "auth_blog_emblem_300.png", size: "200", style: "margin:2em 0"%>
  ```

### v1.0.1 : 2014.6.6 (현충일)

* `posts#index` 뷰 템플릿의 일부 코드 업데이트 함.

  ```html
  <%= link_to icon('eye-open'), post %>&nbsp;
  <% if user_signed_in? %>
    <%= link_to icon('edit'), edit_post_path(post) if post.updatable_by? current_user %>&nbsp;
    <%= link_to icon('trash'), post, method: :delete, data: { confirm: 'Are you sure?' } if post.deletable_by? current_user %>
  <% end %>
  ```

: 사용자 로그인 상태를 체크하는 조건절을 추가함.


### v1.0.0 :  2014.6.6 (현충일)

* 책의 전체 내용을 정리하고 첫번째 공식 버전 릴리스함.


