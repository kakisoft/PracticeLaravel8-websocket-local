## プロジェクト作成
composer create-project --prefer-dist laravel/laravel laravel-websocket
composer create-project --prefer-dist  "laravel/laravel=8.*" my-laravel-app

composer install

______________________________________________________________________________________________
______________________________________________________________________________________________
______________________________________________________________________________________________
https://beyondco.de/docs/laravel-websockets/getting-started/installation


composer require beyondcode/laravel-websockets

php artisan vendor:publish --provider="BeyondCode\LaravelWebSockets\WebSocketsServiceProvider" --tag="migrations"

⇒
my-laravel-app\database\migrations\0000_00_00_000000_create_websockets_statistics_entries_table.php
が生成される

```
PS C:\kaki\__myrepo__\lara-web-socket\my-laravel-app> php artisan vendor:publish --provider="BeyondCode\LaravelWebSockets\WebSocketsServiceProvider" --tag="migrations"
Unable to locate publishable resources. 
Publishing complete.
```


## my-laravel-app\vendor\beyondcode\laravel-websockets\config\websockets.php
本当は、「 \config\websockets.php 」に存在するべきファイル？


```
php artisan serve
```


試しに↑のファイルを移動させてみた
```
ErrorException (E_WARNING)
require(C:\kaki\__myrepo__\lara-web-socket\my-laravel-app\vendor\beyondcode\laravel-websockets\config\websockets.php): failed to open stream: No such file or directory
```

次。  

https://beyondco.de/docs/laravel-websockets/basic-usage/pusher  

```
composer require pusher/pusher-php-server "~3.0"
```


## .env
```
BROADCAST_DRIVER=log
　　↓
BROADCAST_DRIVER=pusher
```


## config\broadcasting.php
https://beyondco.de/docs/laravel-websockets/basic-usage/pusher
```php
        'pusher' => [
            'driver' => 'pusher',
            'key' => env('PUSHER_APP_KEY'),
            'secret' => env('PUSHER_APP_SECRET'),
            'app_id' => env('PUSHER_APP_ID'),
            'options' => [
                'cluster' => env('PUSHER_APP_CLUSTER'),
                'useTLS' => true,
            ],
        ],
```
　　　↓
```php
        'pusher' => [
            'driver' => 'pusher',
            'key' => env('PUSHER_APP_KEY'),
            'secret' => env('PUSHER_APP_SECRET'),
            'app_id' => env('PUSHER_APP_ID'),
            'options' => [
                'cluster' => env('PUSHER_APP_CLUSTER'),
                'useTLS' => true,
                'host' => '127.0.0.1',
                'port' => 6001,
                'scheme' => 'http'
            ],
        ],
```


## Laravel Echo
クライアントサイドで使用するライブラリ
```
npm install laravel-echo pusher-js
```

## _
https://beyondco.de/docs/laravel-websockets/basic-usage/pusher  

```js
import Echo from "laravel-echo"

window.Pusher = require('pusher-js');

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: 'your-pusher-key',
    wsHost: window.location.hostname,
    wsPort: 6001,
    forceTLS: false,
    disableStats: true,
});
```

## resources\js\bootstrap.js
自動生成されている。コメントアウトして使用。一部、設定を追加。
```js
import Echo from 'laravel-echo';

window.Pusher = require('pusher-js');

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: process.env.MIX_PUSHER_APP_KEY,
    cluster: process.env.MIX_PUSHER_APP_CLUSTER,
    forceTLS: true
});
```
　　　↓
```js
import Echo from 'laravel-echo';

window.Pusher = require('pusher-js');

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: process.env.MIX_PUSHER_APP_KEY,
    wsHost: window.location.hostname,
    wsPort: 6001,
    disableStats: true,
});
```

以下を削除している
```
    cluster: process.env.MIX_PUSHER_APP_CLUSTER,
    forceTLS: false,
```


```
npm run watch
```
```
php artisan serve
```
```
php artisan websocket:serve
```

## イベント作成
```
php artisan make:event WebsocketDemoEvent
```

## app\Events\WebsocketDemoEvent.php
```php
class WebsocketDemoEvent implements ShouldBroadcast
{
    public $somedata;

    public function __construct($somedata)
    {
        $this->somedata = $somedata;
    }

    public function broadcastOn()
    {
        // return new PrivateChannel('channel-name');
        return new Channel('DemoChannel');
    }
```



## resources\js\bootstrap.js
```js
window.Echo.channel('DemoChannel')
.listen('WebsocketDemoEvent', (e)=> {
    console.log(e)
})
```
