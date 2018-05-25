# sysvale-laravel-notifications
Tutorial simples de notificações no Laravel 5.6

## Pré-requisitos
* [Ubuntu-PHPS Docker Image](https://github.com/lissonpsantos2/dockerfiles/tree/master/ubuntu-PHPS)
  * `$ docker pull lissonpsantos2/ubuntu-phps`
  * `$ docker run -v /caminho/completo/project:/home/project-folder -p 8080:80 -it lissonpsantos2/ubuntu-phps`
  * `$ selectphp 7.1`
* Criar uma conta no [Pusher](https://pusher.com/)
  * Criar um canal

## Procedimento

### Setup do projeto
> No console do Docker
1. `$ cd /home/project-folder`
2. `$ mkdir notifications-class`
3. `$ cd notifications-class`
4. `$ composer create-project --prefer-dist laravel/laravel notifications`
5. `$ cd notifications`
6. `$ npm install --save laravel-echo pusher-js cross-env`
7. `$ composer require pusher/pusher-php-server`
8. Teste a instalação
   1. `$ apachelinker /home/project-folder/notifications-class/notifications/public`
9. Configure o moloquent
   1. [jenssegers/laravel-mongodb](https://github.com/jenssegers/laravel-mongodb)

> No seu editor
1. Atualize o seu arquivo `.env` utilizando as credenciais de sua base de dados e com as credenciais do canal criado no Pusher


   ```
    DB_CONNECTION=mongodb
    DB_HOST=10.0.0.230
    DB_PORT=27017
    DB_DATABASE=notifications
    DB_USERNAME=
    DB_PASSWORD=

    BROADCAST_DRIVER=pusher

    PUSHER_APP_ID=
    PUSHER_APP_KEY=
    PUSHER_APP_SECRET=
    PUSHER_APP_CLUSTER=

    MIX_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
    MIX_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"
   ```
1. Abra o arquivo `/app/user.php`
   * Substitua a linha que contém o `use Authenticatable` por `use Jenssegers\Mongodb\Auth\User as Authenticatable;`
1. Abra o arquivo `/config/broadcasting.php` e em `connections` adicione


   ```php
   'pusher' => [
            'driver' => 'pusher',
            'key' => env('PUSHER_APP_KEY'),
            'secret' => env('PUSHER_APP_SECRET'),
            'app_id' => env('PUSHER_APP_ID'),
            'options' => [
                'cluster' => env('PUSHER_APP_CLUSTER'),
                'encrypted' => true,
            ],
        ],
   ```
1. Abra o arquivo `/config/app.php` e descomente a linha `App\Providers\BroadcastServiceProvider::class`
1. Abra o arquivo `/resources/assets/js/bootstrap.js` e descomente as linhas 47 até 56

> De volta ao console do Docker no diretório do projeto
1. `$ npm run dev`
1. `$ php artisan make:auth`
1. `$ php artisan migrate`
1. `$ php artisan make:notification StatusLiked`

### Configurando a notificação
> Voltando ao seu editor de texto
1. Abra o arquivo `/app/notification/StatusLiked.php`
   1. Adicione a linha `use Illuminate\Notifications\Messages\BroadcastMessage;` no início do aquivo
   1. Modifique o construtor da classe
   
   
      ```php
      public function __construct($username)
    	 {
        	$this->username = $username;
        	$this->message = "{$username} liked your status";
    	 }
      ```
   1. Modifique o método `via`
   
   
      ```php
       public function via($notifiable)
    	  {
         	return ['broadcast'];
    	  }
      ```
   1. Adicione o método `toBroadcast`
   
   
      ```php
      public function toBroadcast($notifiable)
    	 {
        	 return new BroadcastMessage([
       		     'message'=>$this->message,
        	     'user'=>auth()->user()
        	 ]);
    	 }
      ```
### Editando o layout
1. Abra o arquivo `/resources/view/layout/app.blade.php`
   1. Adicione ao início do seu arquivo as linhas abaixo, o jquery deve ser colocado antes do `<script src="{{ asset('js/app.js') }}" defer></script>` e o bootstrap-notifications antes do `<link href="{{ asset('css/app.css') }}" rel="stylesheet">`
   
   
      ```html
      <link rel="stylesheet" type="text/css" href="/css/bootstrap-notifications.min.css">
      <script src="//cdnjs.cloudflare.com/ajax/libs/jquery/2.1.4/jquery.min.js" defer></script>
      ```
   1. Troque a sua tag `<body>` por
   
   
      ```php
      @if (Auth::check())
       <body data-user-id="{{ Auth::user()->id }}">
      @else
       <body>
      @endif
      ```
   1. Dentro da tag `<ul class="navbar-nav mr-auto"></ul>` adicione
   
   
      ```html
      <li class="nav-item dropdown dropdown-notifications">
          <a href="#notifications-panel" class="nav-link dropdown-toggle" data-toggle="dropdown">
            Notifications (<span data-count="0" class="notif-count">0</span>)
          </a>

          <ul class="dropdown-menu">
          </ul>
      </li>
      ```
1. Abra o arquivo `/resources/assets/sass/app.scss` e adicione a classe


   ```css
   .media-body {
     padding-left: 5.6px;
   }
   ```
1. Abra novamente o arquivo `/resources/assets/js/app.js` e adicione o código seguinte


   ```javascript
   var notificationsWrapper = $('.dropdown-notifications');
   var notificationsToggle = notificationsWrapper.find('a[data-toggle]');
   var notificationsCountElem = notificationsToggle.find('span[data-count]');
   var notificationsCount = parseInt(notificationsCountElem.data('count'));
   var notifications = notificationsWrapper.find('ul.dropdown-menu');

   if (notificationsCount <= 0) {
       notificationsWrapper.hide();
   }

   var updateNotifications = function (data) {
       var existingNotifications = notifications.html();
       var avatar = Math.floor(Math.random() * (71 - 20 + 1)) + 20;
       var newNotificationHtml = `
           <li class="notification active">
               <div class="media">
                   <div class="media-left">
                   <div class="media-object">
                       <img src="https://api.adorable.io/avatars/71/`+ avatar + `.png" class="img-circle" alt="50x50" style="width: 50px; height: 50px;">
                   </div>
                   </div>
                   <div class="media-body">
                   <strong class="notification-title">`+ data.message + `</strong>
                   <!--p class="notification-desc">Extra description can go here</p-->
                   <div class="notification-meta">
                       <small class="timestamp">about a minute ago</small>
                   </div>
                   </div>
               </div>
           </li>
           `;

       notifications.html(newNotificationHtml + existingNotifications);
       notificationsCount += 1;
       notificationsCountElem.attr('data-count', notificationsCount);
       notificationsWrapper.find('.notif-count').text(notificationsCount);
       notificationsWrapper.show();
   };
   
   if($('body').data('user-id')){
      Echo.private('App.User.' + $('body').data('user-id'))
          .notification((notification) => {
              console.log(notification.type);
              updateNotifications(notification);
      });
   }
   ```
> Faça o download do [bootstrap-notifications](https://skywalkapps.github.io/bootstrap-notifications/) e extraia o arquivo dentro de `/public/css`


### Acionando a notificação
1. Abra o arquivo `/routes/web.php`e adicione a seguinte rota


   ```php
   Route::get('/notify', function(){
     Auth::user()->notify(new \App\Notifications\StatusLiked('Someone'));
     // Notification::send(Auth::user(), new \App\Notifications\StatusLiked('Someone'));
     return "Notification has been sent!";
   });
   ```
> De volta ao console do Docker
* Execute novamente o comando `npm run dev`

## Teste
1. Com seu navegador, acesse o endereço `/register` e crie um novo usuário
2. Em outra aba/janela, acesse o endereço `/notify` e verifique se a notificação ocorreu na barra de navegação

## Fontes
* [WebDevMatics - YouTube](https://youtu.be/i6Rdkv-DLwk)
* [WebDevMatics - GitHub](https://github.com/webdevmatics/webdevforum)
* [Pusher](https://pusher.com/tutorials/web-notifications-laravel-pusher-channels)
