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


