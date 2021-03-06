API
========
High performance api service

单服开10+Worker，轻松支撑C100K请求

Windows下安装PHP7
========
```
下载最新PHP7.x http://windows.php.net/download/
解压到任意目录，比如F:\php
将PHP的安装路径F:\php加入PATH 环境变量
进入PHP安装目录（例如 F:\php）。找到php.ini-development文件并复制一份到当前目录，重命名为php.ini
用编辑器打开 php.ini 文件，修改以下配置：
去掉extension_dir = "ext"前面的分号（738 行左右）
去掉extension=php_curl.dll前面的分号（893 行左右）
去掉extension=php_mbstring.dll前面的分号（903 行左右）
去掉extension=php_openssl.dll前面的分号（907 行左右）
去掉extension=php_pdo_mysql.dll前面的分号（909 行左右）
```

Windows下安装Composer
========
```
下载并执行安装
https://getcomposer.org/Composer-Setup.exe
使用中国镜像
composer config -g repo.packagist composer https://packagist.phpcomposer.com
```

Mac下安装PHP7
========
```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew update
brew untap homebrew/php
brew uninstall php
brew upgrade
brew clenaup
brew prune
brew install autoconf
brew install php
```

Mac下安装MySQL/Redis
========
```
brew install mysql
brew services start mysql
brew install redis
brew services start redis
git clone https://github.com/phpredis/phpredis.git
cd phpredis && phpize && ./configure && make && make install
echo "extension=redis.so" > /usr/local/etc/php/7.2/conf.d/redis.ini
```

Mac下安装Composer
========
```
安装composer
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
composer config -g repo.packagist composer https://packagist.phpcomposer.com
```

Linux下安装PHP7
========
```
yum install php70w php70w-cli php70w-common php70w-gd php70w-ldap php70w-mbstring php70w-mcrypt php70w-mysql php70w-pdo php70w-fpm php70w-pecl-redis -y
```
安装Workerman的pcntl和posix扩展、event或者libevent扩展：http://doc.workerman.net/315116

内核参数调优：http://doc.workerman.net/315302

Linux下安装Composer
========
```
更新yum安装包
rpm -Uvh http://mirror.webtatic.com/yum/el6/latest.rpm
安装composer
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
composer config -g repo.packagist composer https://packagist.phpcomposer.com
```

示例代码
========
```php
/////////////////////////////////// http
<?php
require_once __DIR__ . '/vendor/autoload.php';
require_once __DIR__ . '/app_http.php';
require_once __DIR__ . '/auth.php';

$app = new App("http://0.0.0.0:2345");
$app->count = 4;
$app->name = 'http';

$app->onWorkerStart = function($worker) {
	require_once __DIR__ . '/db.php';
	require_once __DIR__ . '/redis.php';
};

$app->get('/', function($req) {
	return "666";
});

$app->post('/', function($req) {
	return "666";
});

$app->get('/db', function($req) {
	$all_tables = MyDB::get_tables();
	MyRedis::set_tables($all_tables);
	return $all_tables;
});

$app->get('/tables', function($req) {
	$all_tables = MyRedis::get_tables();
	return $all_tables;
});

$app->before('/api', function($req) {
	return Auth::verify_sign($req->params);
});

$app->get('/api/test', function($req) {
	$data = array('name'=>'dad');
	return $data;
});

App::runAll();

/////////////////////////////////// websocket
<?php
require_once __DIR__ . '/vendor/autoload.php';
require_once __DIR__ . '/app_ws.php';

$wsapp = new WSApp("websocket://0.0.0.0:2000/");
$wsapp->count = 4;
$wsapp->name = 'ws';

$wsapp->on('/', function($params) {
	return array('code' => '200');
});

$wsapp->on('/api', function($params) {
	if ($params->action == 'subscribe') {
		return $params;
	}
	return array('code' => '404');
});

WSApp::runAll();
```

启动服务
========
```
php http.php start
php ws.php start
```
压力测试
```
ab -n 1000000 -c 1000 -k http://localhost:2345/
ab -n 1000000 -c 1000 -k "http://localhost:2345/api/test?key=abcdefg1234567&sign=c0f3b42d494891b203023bfc3d50af533b50b05de89b57b2f4ec50f357ba8fc0b333444be0d02b608820ba5b7de9155b5c7bb7dad9d1bb38c962322569c5b92b"
```
