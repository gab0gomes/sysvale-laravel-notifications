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
### No console do Docker
1. `$ cd /home/project-folder`
2. `$ mkdir notifications-class`
3. `$ cd notifications-class`
4. `$ composer create-project --prefer-dist laravel/laravel notifications`
5. `$ cd notifications`
6. `$ cd npm install --save laravel-echo pusher-js cross-env`
7. `$ composer require pusher/pusher-php-server`
8. Teste a instalação
   1. `$ apachelinker /home/project-folder/notifications-class/notifications/public`
9. Configure o moloquent
   1. [jenssegers/laravel-mongodb](https://github.com/jenssegers/laravel-mongodb)

### No seu editor
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
1. Abra o arquivo `user.php`
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
1. Abra o arquivo `/resources/js/bootstrap.js` e descomente as linhas 47 até 56
1. Abra o arquivo `/resources/js/app.js` e adicione
   ```javascript
   Echo.private('App.User.' + 'userId')
    .notification((notification) => {
      console.log(notification.type);
    });
   ```
### De volta ao console do Docker no diretório do projeto
1. `$ npm run dev`
1. `$ php artisan make:auth`
1. `$ php artisan migrate`
1. `$ php artisan make:notification StatusLiked`

### No seu editor de texto
1. Abra o arquivo `/app/notification/StatusLiked.php`
   1. Adicione a linha `use Illuminate\Notifications\Messages\BroadcastMessage;` no início do aquivo
   1. Modifique o construtor da classe
      ```php
      public function __construct()
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
        	     'user'=>$username
        	 ]);
    	 }
      ```
1. Abra o arquivo `/resources/view/layout/app.blade.php`
   1. Adicione ao início do seu arquivo as linhas
      ```html
      <link rel="stylesheet" type="text/css" href="/css/bootstrap-notifications.min.css">
      <script src="//cdnjs.cloudflare.com/ajax/libs/jquery/2.1.4/jquery.min.js" defer></script>
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
> Faça o download do [bootstrap-notifications](https://skywalkapps.github.io/bootstrap-notifications/) e extraia o arquivo dentro de `/public/css`

1. Abra o arquivo `/resources/assets/app.scss` e adicione a classe
   ```css
   .media-body {
    padding-left: 5.6px;
   }
   ```
