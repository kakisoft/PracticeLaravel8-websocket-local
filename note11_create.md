## 参考動画　　
https://www.youtube.com/watch?v=pIGy7-7gGXI

## Laravel Websockets
https://beyondco.de/docs/laravel-websockets/getting-started/installation

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

