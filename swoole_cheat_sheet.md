# 使用内置Http异步客户端
### [SSL支持](https://wiki.swoole.com/wiki/page/678.html)
```php
$cli = new swoole_http_client('127.0.0.1', 80, true);
//如果服务器需要提供SSL证书
$cli->set(array(
    'ssl_cert_file' => $certFile,
    'ssl_key_file' => $keyFile,
));
```

### [Socks5代理](https://wiki.swoole.com/wiki/page/678.html)
```php
$cli->set(array(
    'socks5_host'     =>  '192.168.1.100',
    'socks5_port'     =>  1080,
    'socks5_username' => 'username', //用户名和密码为可选项
    'socks5_password' => 'password',
));
```

### [上传文件](https://wiki.swoole.com/wiki/page/678.html)
```php
$cli = new swoole_http_client('127.0.0.1', 80);

$cli->addFile(__DIR__.'/post.data', 'post');
$cli->addFile(dirname(__DIR__).'/test.jpg', 'debug');

$cli->post('/dump2.php', array("xxx" => 'abc', 'x2' => 'rango'), function ($cli) {
    echo $cli->body;
    $cli->close();
});
```


 ### [发起GET请求](https://wiki.swoole.com/wiki/page/534.html)

```php
$cli = new swoole_http_client('127.0.0.1', 80);

$cli->setHeaders([
    'Host' => "localhost",
    "User-Agent" => 'Chrome/49.0.2587.3',
    'Accept' => 'text/html,application/xhtml+xml,application/xml',
    'Accept-Encoding' => 'gzip',
]);

$cli->get('/index.php', function ($cli) {
    echo "Length: " . strlen($cli->body) . "\n";
    echo $cli->body;
});

```
 ### [发起POST请求](https://wiki.swoole.com/wiki/page/678.html)

```php
$cli = new swoole_http_client('127.0.0.1', 80);
$cli->setHeaders(['User-Agent' => "swoole"]);

$cli->post('/dump.php', array("test" => 'abc'), function ($cli) {
    echo $cli->body;
});
```

###  [发起WebSocket握手请求，并将连接升级为WebSocket](https://wiki.swoole.com/wiki/page/536.html)
```php
$cli = new swoole_http_client('127.0.0.1', 9501);

$cli->on('message', function ($_cli, $frame) {
    var_dump($frame);
});

$cli->upgrade('/', function ($cli) {
    echo $cli->body;
    $cli->push("hello world");
});

$client->set([
    'websocket_mask' => true,
    'ssl_host_name' => 'www.yourdomain.com',
]);
$client->setHeaders([
    'Host' =>  'www.yourdomain.com',
    'UserAgent' => 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36'
]);
```

# 服务器
### Web服务器
```php
$http = new swoole_http_server("0.0.0.0", 9501);

$http->on('request', function ($request, $response) {
    var_dump($request->get, $request->post);
    $response->header("Content-Type", "text/html; charset=utf-8");
    $response->end("<h1>Hello Swoole. #".rand(1000, 9999)."</h1>");
});

$http->start();
```

### WebSocket服务器
```php
//创建websocket服务器对象，监听0.0.0.0:9502端口
$ws = new swoole_websocket_server("0.0.0.0", 9502);

//监听WebSocket连接打开事件
$ws->on('open', function ($ws, $request) {
    var_dump($request->fd, $request->get, $request->server);
    $ws->push($request->fd, "hello, welcome\n");
});

//监听WebSocket消息事件
$ws->on('message', function ($ws, $frame) {
    echo "Message: {$frame->data}\n";
    $ws->push($frame->fd, "server: {$frame->data}");
});

//监听WebSocket连接关闭事件
$ws->on('close', function ($ws, $fd) {
    echo "client-{$fd} is closed\n";
});

$ws->start();
```

### TCP服务器
```php
//创建Server对象，监听 127.0.0.1:9501端口
$serv = new swoole_server("127.0.0.1", 9501); 

//监听连接进入事件
$serv->on('connect', function ($serv, $fd) {  
    echo "Client: Connect.\n";
});

//监听数据接收事件
$serv->on('receive', function ($serv, $fd, $from_id, $data) {
    $serv->send($fd, "Server: ".$data);
});

//监听连接关闭事件
$serv->on('close', function ($serv, $fd) {
    echo "Client: Close.\n";
});

//启动服务器
$serv->start();
```
### UDP服务器
```php
//创建Server对象，监听 127.0.0.1:9502端口，类型为SWOOLE_SOCK_UDP
$serv = new swoole_server("127.0.0.1", 9502, SWOOLE_PROCESS, SWOOLE_SOCK_UDP); 

//监听数据接收事件
$serv->on('Packet', function ($serv, $data, $clientInfo) {
    $serv->sendto($clientInfo['address'], $clientInfo['port'], "Server ".$data);
    var_dump($clientInfo);
});

//启动服务器
$serv->start(); 
```

### 执行异步任务

```php
$serv = new swoole_server("127.0.0.1", 9501);

//设置异步任务的工作进程数量
$serv->set(array('task_worker_num' => 4));

$serv->on('receive', function($serv, $fd, $from_id, $data) {
    //投递异步任务
    $task_id = $serv->task($data);
    echo "Dispath AsyncTask: id=$task_id\n";
});

//处理异步任务
$serv->on('task', function ($serv, $task_id, $from_id, $data) {
    echo "New AsyncTask[id=$task_id]".PHP_EOL;
    //返回任务执行的结果
    $serv->finish("$data -> OK");
});

//处理异步任务的结果
$serv->on('finish', function ($serv, $task_id, $data) {
    echo "AsyncTask[$task_id] Finish: $data".PHP_EOL;
});

$serv->start();
```
