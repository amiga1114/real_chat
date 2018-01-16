### 1. 실시간 채팅을 위한 기본 구현

> 소켓을 통해 구현되어 있다. 간단하게 생각하자 !
>
> 1. 프로젝트 만들기
>
> ```
> $ rails new real_chat --skip-bundle
>
> ```
>
> 1. Gemfile 추가하기
>
> ```
> gem 'rails_db'
> gem 'awesome_print'
> gem 'pry-rails'
> ```
>
> - bundle
>
> ```
> $ bundle install
>
> ```
>
> 1. scaffold Post 만들기
>
> ```
> $ rails g scaffold Post title content:text
>
> ```
>
> 1. Comment 모델 만들기
>
> ```
> $ rails g model Comment post_id:integer content:text
>
> ```
>
> - Post : Comment -> 1 : N 만들기
>
> ```
> # app/models/post.rb
> class Post < ActiveRecord::Base
>  has_many :comments
> end
> # app/models/comment.rb
> class Comment < ActiveRecord::Base
>  belongs_to :post
> end
> ```
>
> 1. 마이그레이션 해주기
>
> ```
> $ rake db:migrate
>
> ```
>
> 1. 댓글 입력을 위해 show.html.erb 수정하기
>
> ```
> <%= link_to 'Edit', edit_post_path(@post) %> |
> <%= link_to 'Back', posts_path %>
>
> <%= form_tag "/posts/#{@post.id}/add_comment", method: :get do %>
>  <%= text_field_tag :content %>
>  <%= submit_tag :submit %>
> <% end %>
> ```
>
> 1. routes.rb 수정하기
>
> ```
> get '/posts/:id/add_comment' => 'posts#add_comment'
> ```
>
> 1. Post 컨트롤러 수정하기
>
> ```
> def add_comment
>  Comment.create(
>    post_id: params[:id],
>    content: params[:content]
>  )
>  redirect_to :back
> end
> ```
>
> 1. show 화면에서 댓글 보여주기
>
> ```
> <p>--------------댓글목록---------------</p>
> <% @post.comments.each do |comment| %>
>  <p><%= comment.content %></p>
> <% end %>
> ```

### 2. 레일즈에서 자바스크립트 사용하기

> #### remote : true -> 서버에 보내기만하고 응답을 새로받지 않음. 즉 페이지 리로드 안함
>
> ajax 를 대체하는 코드를 짜보자 !
>
> 1. posts_controller.rb 수정하기
>
> ```
> def add_comment
>  Comment.create(
>    post_id: params[:id],
>    content: params[:content]
>  )
> end
> ```
>
> 1. /app/views/add_comment.js.erb 파일 만들기
>
> ```
> $('#comments').append("<p><%= @comments.content %></p>")
> ```
>
> 1. post_controller.rb 수정하기
>
> ```
> def add_comment
>  @comments = Comment.new(
>    post_id: params[:id],
>    content: params[:content]
>  )
>  @comments.save
> end
> ```
>
> `Server` - `Channel` - > `User` 구독
>
>  - > `User` 구독
>
> ### 서버에 상태를 저장해두고 변화에 대한 응답을 하는애 '소켓 또는 채널'
>
> - Actioncable, Pusher을 이용하여 소켓 또는 채널을 만들 수 있음
> - 하지만 우리는 Pusher로 사용하겠음 !
>
> 1. Gemfile 추가하기
>
> ```
> gem 'pusher'
> gem 'devise'
> ```
>
> - gem 설치하기
>
> ```
> $ bundle install
>
> ```
>
> 1. config/initalizers/pusher.rb 추가하기
>
> ```
> Pusher.app_id = 'app_id'
> Pusher.key = 'user key'
> Pusher.secret = 'user secret key'
> Pusher.cluster = 'user cluster'
> Pusher.logger = Rails.logger
> Pusher.encrypted = true
> ```
>
> 1. posts_controller.rb 수정하기
>
> ```
> def hello_world
>  Pusher.trigger('my-channel', 'my-event', {
>    message: 'hello world'
>  })
> end
> ```
>
> 1. views/layout/application.html.erb 수정하기
>
> ```
> <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true %>
> <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
> <script src="https://js.pusher.com/4.1/pusher.min.js"></script>
> <%= csrf_meta_tags %>
> ```
>
> 1. views/posts/hello.html.erb 수정하기
>
> ```
> <script>
>  // Enable pusher logging - don't include this in production
>  Pusher.logToConsole = true;
>
>  var pusher = new Pusher('user key', {
>    cluster: 'ap1',
>    encrypted: true
>  });
>
>  var channel = pusher.subscribe('my-channel');
>  channel.bind('my-event', function(data) {
>  	alert(data.message);
>  });
> </script>
> ```
>
> ### 이제 본격적으로 우리 코드에 적용해보자
>
> 1. posts_controller.rb 수정하기
>
> ```
> def talk
>  @msg = params[:msg]
>  # render :talk
> end
> ```
>
> 1. hello.html.erb 수정하기
>
> ```
> <h1>pusher 테스트</h1>
> <%= form_tag '/talk', method: :get, remote: true do %>
>  <%= text_field_tag :msg %>
>  <%= submit_tag :submit %>
> <% end %>
> <div id="talks">
>  <p></p>
> </div>
>
> <script>
>  // Enable pusher logging - don't include this in production
>  console.log("Pusher 써볼게")
>  Pusher.logToConsole = true;
>
>  var pusher = new Pusher('04c565922ecaf9fca27c', {
>    cluster: 'ap1',
>    encrypted: true
>  });
>
>  var channel = pusher.subscribe('real_chat');
>  channel.bind('hello', function(data) {
>    // alert(data.message);
>  });
> </script>
> ```
>
> 1. talk.js.erb 추가하고 수정하기
>
> ```
> $('#talks').append("<p><%= @msg %></p>")
> ```
>
> 1. routes.rb 수정하기
>
> ```
> get '/talk' => 'posts#talk'
> ```
>
> 1. talk 모델 만들기
>
> ```
> $ rails g model Talk message
>
> ```
>
> - 마이그레이트 해주기
>
> ```
> $ rake db:migrate
>
> ```
>
> 1. posts_controller.rb 수정하기
>
> ```
> def talk
>  @talk = Talk.new(
>    message: params[:msg]
>  )
>  @talk.save
>  Pusher.trigger('real_chat', 'hello', { #채널이름, #이벤트 이름
>    message: params[:msg]
>  })
>  # @msg = params[:msg]
>  # render :talk
> end
>
> def hello
>  @talks = Talk.all.reverse
>  # Pusher.trigger('real_chat', 'hello', { #채널이름, #이벤트 이름
>  #   message: 'hello world'
>  # })
> end
> ```
>
> 1. hello.html.erb 수정하기
>
> ```
> <%= form_tag '/talk', method: :get, remote: true do %>
>  <%= text_field_tag :msg %>
>  <%= submit_tag :submit %>
> <% end %>
> <div id="talks">
>  <% @talks.each do |talk| %>
>    <p><%= talk.message %></p>
>  <% end %>
> </div>
> ```
>
> 1. talk.js.erb 수정하기 // hello.html.erb에서 작성한 <script></script> 내용들을 다 옮기자 !
>
> ```
> console.log("Pusher 써볼게")
> Pusher.logToConsole = true;
>
> var pusher = new Pusher('04c565922ecaf9fca27c', {
>  cluster: 'ap1',
>  encrypted: true
> });
>
> var channel = pusher.subscribe('real_chat');
> channel.bind('hello', function(data) {
>  // alert(data.message);
>  $('#talks').prepend("<p>" + data.message + "</p>")
> });
> ```