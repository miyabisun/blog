---
title: 【PHP】エラーログを閲覧
date: 2013-06-19 10:25:36
tags: PHP
id: php-error-log
---

## PHPのエラーログを監視する

PHPのエラーは常に画面へ出力されるとは限らない。
例えばコンソールからPHPファイルを実行した場合、
php.iniの設定によってはincludeしたファイルの関数衝突によるFatalエラーはコンソールに表示されない。

<!-- more -->

## PHPのログファイルって何処に吐き出される？

php.ini内で`error_log = "xxx"`と記載したxxxにエラーログが吐出される
<http://jp.php.net/manual/ja/errorfunc.configuration.php#ini.error-log>

これはphpinfoで確認できるので、
コンソールなら以下のコマンド打つだけで取得出来て楽
後はそのままtailコマンドで監視すればいい

```PHP
# php -i | grep error_log
error_log => /var/log/php_error.log => /var/log/php_error.log
# tail -f /var/log/php_error.log
```

## エラー出ないぞ

Apacheを通した場合、ApacheのエラーログにPHPのエラーが吐き出される場合がある
CentOSの場合、「/etc/httpd/conf/httpd.conf」ファイル内に記載すると、
その中の記述に従ったディレクトリに作成される

```
# vi /etc/httpd/conf/httpd.conf
<VirtualHost *:80>
    ServerName   hogehoge.com:80
    ServerAdmin  test@hogehoge.com
    DocumentRoot "/var/www/html/"
    ErrorLog     logs/php.err_log
    CustomLog    logs/php.acc_log combined
    <Directory "/var/www/html/">
        Options Indexes FollowSymLinks
        AllowOverride All
        Order allow,deny
        Allow from all
    </Directory>
</VirtualHost>

# tail -f /etc/httpd/logs/php.err_log
```

上記の`ErrorLog     logs/php.err_log`に注目、
デフォルトだとこんな感じの設定になっていると思うが、カレントディレクトリは`/etc/httpd`になっているので

エラーログは`/etc/httpd/logs/php.err_log`になる。
後は同じようにtailコマンドで監視すればよい。

