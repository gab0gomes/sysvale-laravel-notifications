# sysvale-laravel-notifications
Tutorial simples de notificações no Laravel 5.6

## Pré-requisitos
* [Ubuntu-PHPS Docker Image](https://github.com/lissonpsantos2/dockerfiles/tree/master/ubuntu-PHPS)
  * `$ docker pull lissonpsantos2/ubuntu-phps`
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
1. Abra o arquivo `\config\broadcasting.php` e em `connections` adicione
   ```
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
1. Abra o arquivo `\config\app.php` e descomente a linha `App\Providers\BroadcastServiceProvider::class`
1. Abra o arquivo `\resources\js\bootstrap.js` e descomente as linhas 47 até 56
1. Abra o arquivo `\resources\js\app.js` e adicione
   ```
   Echo.private('App.User.' + 'userId')
    .notification((notification) => {
      console.log(notification.type);
    });
   ```
   
