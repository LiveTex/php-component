**Установка**

Компонент основан на фреймворке Laravel 4.2 с отключенными лишними модулями

```bash
wget https://github.com/LiveTex/php-component/archive/1.0.tar.gz
tar -xzf php-component-1.0.tar
cd php-component-1.0
composer install --no-scripts
composer dump-autoload
cd ..
mv php-component-1.0 project_name
```

**Настройка nginx**

```nginx
server {
    server_name offline;

    root /path_to/project_name/public;
    index index.php;

    location / {
        index index.php;
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME /path_to/project_name/public$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```

**Пример приложения**

Необходимые скомпилированные thrift файлы необходимо скопировать в папку app/thrift

```bash
cp -Rf <thrift>/gen-php/ <application>/app/thrift/
```

Слешы на конце путей в коменде выше важны, необходимо перенести содержимое папки gen-php в app/thrift, так же просто можно скомпилировать thrift файлы с "--out=<application>/app/thrift/".

**Простой пример**

Создаем файл описания для thrift сервиса app/model/ServiceConfig.php, на примере сервиса Offline. 
Заполняем namespace и service соотвествующими значениями.

```php
<?php

class ServiceConfig extends \Platform\EndpointConfig
{
    public $namespace = 'offline';
    public $service = 'Offline';
}
```

Теперь нам нужен Handler для обработки входящих команд. Для этого, создаем необходимый файл в app/model с названием Handler(ServiceName).php, где ServiceName это имя сервиса из конфига выше. Класс должен импрементировать интерфейс сервиса, сгеренированного thrift'ом. Данный файл по сути является контроллером, по этому логику следует реализовавывать в отдельных моделях под каждый метод.

```php
class HandlerOffline implements \offline\OfflineIf
{
    // Имплементируем все методы интерфейса, но в данном примере нас 
    // интересует только Ping(), по этому описан только он.
    
    ...
    
    public function Ping()
    {
        return rand(0,1) == 1;
    }

}
```

Создаем точку входа. Добавляем в файл app/routes.php:

```php
Route::post('/', 'IndexController@postRequest');
```

Открываем app/controllers/IndexController.php и дописываем делегирование входящих thrift пакетов в процессор сервиса.

```php
...
public function postRequest()
{
    $config = new ServiceConfig();

    $endpoint = new \Platform\Endpoint($config);

    $endpoint->process();
}
...
```

Готово. Наша точка входа способна принимать thrift пакеты, обрабатывать их и отдавать результат.

**Тестируем результат**

Теперь попробуем обратиться к нашей точке входа с помощью thrift и получить результат из приложения.
Можно для этого можно использовать дефолтную точку входа в приложение "(SERVER_NAME)/", маршрут уже описан, так что надо только переписать отвечающий за маршрут метод в файле IndexController, метод Index. В примере SERVER_NAME надо заменить на то, которое используется в этом приложении в nginx.

```php
...
public function Index()
{
    $config = new ServiceConfig();

    $endpoint = new \Platform\Endpoint($config);

    $client = $endpoint->getClient('SERVER_NAME', 80, '/');

    var_dump($client->Ping());
}
...
```

Проверяем через консоль:

```bash
➜ curl http://SERVER_NAME/
bool(true)
➜ curl http://SERVER_NAME/
bool(false)
```
Если результат примерно такой, т.е. в случайном порядке выдает true или false, значит все работает и можно приступать к реализации логики более сложных методов.

**Литература**

Допументация по работе с фремворком Laravel доступер по адресу http://laravel.com/docs/4.2
