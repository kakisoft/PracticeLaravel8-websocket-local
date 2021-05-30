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

```

## instal
```
cd .\laravel-websocket\

composer install

composer require beyondcode/laravel-websockets

php artisan vendor:publish --provider="BeyondCode\LaravelWebSockets\WebSocketsServiceProvider" --tag="migrations"
```







