**NOTE**: If you're reading this tutorial, make sure you've completed the [LEMP stack tutorial](https://github.com/HCDigitalScholarship/documentation/blob/master/lemp_stack.md) up to step 4.

PHP is an open source web scripting language that is widely use to build dynamic webpages. You only need to install PHP if you're using Omeka, Drupal or another PHP-bases content management system. Otherwise, there's no need and you can skip to the next step.

To install PHP, open terminal and type in this command.

```
$ sudo apt-get install php5-fpm
```

Next, we need to make one small change in the php configuration. Open up `php.ini`:

```
$ sudo vim /etc/php5/fpm/php.ini
```

Find the line, `cgi.fix_pathinfo=1`, and change the 1 to 0.

If this number is kept as 1, the PHP interpreter will do its best to process the file that is as near to the requested file as possible. This is a possible security risk. If this number is set to 0, conversely, the interpreter will only process the exact file path—a much safer alternative.

[This next step may be optional, it was already set when I tried it] We need to make another small change in the php5-fpm configuration. Open up `www.conf`:

```
$ sudo vim /etc/php5/fpm/pool.d/www.conf
```

Find the line, `listen = 127.0.0.1:9000`, and change the `127.0.0.1:9000` to `/var/run/php5-fpm.sock`. 

Restart php-fpm:

```
sudo service php5-fpm restart
```
