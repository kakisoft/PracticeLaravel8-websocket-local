## 参考動画　　
https://www.youtube.com/watch?v=pIGy7-7gGXI

## Laravel Websockets
https://beyondco.de/docs/laravel-websockets/getting-started/installation

_____________________________________________________________________________________
※バックグラウンドで色々動かしている。VSCode を使用する場合、Terminal -> Split Terminal で、ターミナルを分割して実行すると良さげ。

_____________________________________________________________________________________
_____________________________________________________________________________________
_____________________________________________________________________________________
# 手順

## プロジェクト作成
```
composer create-project --prefer-dist laravel/laravel laravel-websocket
composer create-project --prefer-dist  "laravel/laravel=8.*" my-laravel-app

// 今回、こんな感じで
composer create-project --prefer-dist  "laravel/laravel=8.*" laravel-websocket


cd .\laravel-websocket\

composer install
```

## web-socket instal
```
composer require beyondcode/laravel-websockets
```


## migration 用のファイルを作成
```
php artisan vendor:publish --provider="BeyondCode\LaravelWebSockets\WebSocketsServiceProvider" --tag="migrations"

// -> database\migrations\0000_00_00_000000_create_websockets_statistics_entries_table.php  が作成される
```

______________________________________________
# migrate 前の設定
※ローカルで実験する時、SQLite を使用しているため、.env に修正を加えている。

##
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=

    ↓

DB_CONNECTION=sqlite
```

その後、  
database/database.sqlite  
に空のファイルを作成しておく。  

※migration が失敗した場合、php.ini の設定を確認。  
#### 例：php7 - C:\tools\php74\php.ini
```
;extension=pdo_sqlite
　　↓
extension=pdo_sqlite
```

（以下は必須ではない？）
```
;extension=sqlite3
　　↓
extension=sqlite3
```

______________________________________________
## migrate
```
php artisan migrate
php artisan migrate:fresh
```

## key:generate
```
php artisan key:generate
```

## WebSocket configuration file
```
php artisan vendor:publish --provider="BeyondCode\LaravelWebSockets\WebSocketsServiceProvider" --tag="config"

//=> config\websockets.php が生成される
```
config\websockets.php  
にて、以下のような設定が可。  
・接続可能なホスト  
・データサイズの上限  
（詳細はファイルを参照）  

## pusher 用の環境設定
#### .env
```
PUSHER_APP_ID=
PUSHER_APP_KEY=
PUSHER_APP_SECRET=
PUSHER_APP_CLUSTER=mt1

　　　↓

PUSHER_APP_ID=anyID
PUSHER_APP_KEY=anyKEY
PUSHER_APP_SECRET=anySECRET
PUSHER_APP_CLUSTER=mt1
```
※何か適当な値を設定する


## 実行
```
php artisan serve
```

ダッシュボードにアクセス可。  
（ 末尾の URL は、websockets.php の 'path' にて変更可 ）  
※この時点では、ロクな操作ができない。  

http://localhost:8000/laravel-websockets  


## Pusher PHP SDK - install
初期状態ではローカルからしかアクセスできないため、ライブラリを用意する。  
https://beyondco.de/docs/laravel-websockets/basic-usage/pusher  
```
composer require pusher/pusher-php-server "~3.0"
```

その後、.env ファイルを編集
```
BROADCAST_DRIVER=log

　　↓

BROADCAST_DRIVER=pusher
```


ブロードキャストイベントを扱えるようになる


## ブロードキャスト設定を編集

#### broadcasting.php
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

## フロントから使用するためのライブラリを追加
```
npm install
npm install laravel-echo pusher-js
```

## フロントから使用するための設定
https://beyondco.de/docs/laravel-websockets/basic-usage/pusher#usage-with-laravel-echo  

以下を追加
#### resources\js\bootstrap.js
```js
import Echo from 'laravel-echo';

window.Pusher = require('pusher-js');

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: process.env.MIX_PUSHER_APP_KEY,
    cluster: process.env.MIX_PUSHER_APP_CLUSTER,
    wsHost: window.location.hostname,
    wsPort: 6001,
    forceTLS: false,
    disableStats: true,
});
```
cluster, forceTLS, は場合によっては削除してよい？  
あと、「forceTLS」は、環境（or バージョン）によっては、「encrypted」となっている？  


## watch
artisan serve を実行しているターミナルとは、別のターミナルで動作させるのがおすすめ。
```
npm run watch
```


## websocket サーバ起動
別ターミナルを用意した方が動かしやすい。
```
php artisan websocket:serve
```

## ダッシュボード操作
ダッシュボードの操作が可能となる。  

http://localhost:8000/laravel-websockets  


## イベント作成
仮として、名称「WebSocketDemoEvent」で作成。
```
php artisan make:event WebSocketDemoEvent

//=> app\Events\WebSocketDemoEvent.php が作成される
```

イベントを編集。  
 - implements ShouldBroadcast を追加
 - プライベート変数追加  
 - コンストラクタ編集  
 - broadcastOn メソッドを修正  

参考ソース  


以下を追加
#### resources\js\bootstrap.js
```js
window.Echo.channel('DemoChannel')
.listen('WebsocketDemoEvent', (e)=> {
    console.log(e)  // sample
})
```

## js に記述した設定を読み込ませる

例：  
##### resources\views\welcome.blade.php
```html
<script src="js/app.js"></script>
```
csrf token も必要となる。
```html
<meta name="csrf-token" content="{{ csrf_token() }}">
```
以下は無くても行けたけど、付けておいた方がよい？
```html
    <body class="antialiased">
        <div class="relative flex items-top justify-center min-h-screen bg-gray-100 dark:bg-gray-900 sm:items-center py-4 sm:pt-0">

　　　　　↓

    <body class="antialiased">
        <div id="app" class="relative flex items-top justify-center min-h-screen bg-gray-100 dark:bg-gray-900 sm:items-center py-4 sm:pt-0">
```

## 別ウィンドウを開いて操作
http://localhost:8000/  
を複数立ち上げると、それぞれの画面のコンソールにメッセージが表示される。  

ダッシュボードを使えば、チャンネルを指定してメッセージを一斉配信したり、特定のパラメータを渡して実行できる。  


