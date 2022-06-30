# Create Shopify App bridge with Laravel using Osiset on Ubuntu

## Prerequisites

> In order to complete this tutorial, you will need access to the following on an Ubuntu 20.04 server:

- A sudo user on your server
- An Apache2 web server, which you can install with"

```
sudo apt install apache2
```

- PHP 7.4+ version of PHP
- MySQL 5.7+ version of MySQL
- A MySQL database
- MySQL user with privileges
- A Domain Name pointing to the host ip (On DNS like Godaddy, Hostinger, etc.)

### Step 1 — Create the Directory Structure

```
sudo mkdir -p /var/www/example.com/public_html
```

### Step 2 — Grant Permissions

```
sudo chown -R $USER:$USER /var/www/example.com/public_html
```

### Step 3 — Create the Virtual Host File

```
sudo nano /etc/apache2/sites-available/example.com.conf
```

```
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/example.com/public_html
    ServerName example.com
    ServerAlias www.example.com
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

#### Set directory permissions in the Apache2 /etc/apache2/apache2.conf
```
sudo nano /etc/apache2/apache2.conf
```
<Directory /var/www/>
        Options Indexes FollowSymLinks
        AllowOverride False
        Require all granted
</Directory>
```
to make the directory accessible to the web server.
```
<Directory /var/www/>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
</Directory>
```

### Step 4 — Enable the Virtual Host

```
sudo a2ensite example.com.conf
```

### Step 5 — Reload Apache

```
sudo systemctl reload apache2
```

### Step 7 — Create the Index File

```
sudo nano /var/www/example.com/public_html/index.html
```

### Paste the following code into the index.html file

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <h1>Hello World</h1>
</body>
</html>
```

### Step 8 — Test your site with a Browser

```
http://example.com
```

# Install letsencrypt SSL on Ubuntu

## Install Certbot in Ubuntu with PIP

> First, install PIP:

```
sudo apt install python3 python3-venv libaugeas0
```

> Set up a virtual environment:

```
sudo python3 -m venv /opt/certbot/

sudo /opt/certbot/bin/pip install --upgrade pip
```

> Install Certbot on Apache:

```
sudo /opt/certbot/bin/pip install certbot certbot-apache
```

> Install Certbot on NGINX:

```
sudo /opt/certbot/bin/pip install certbot certbot-nginx
```

> Create a symlink to ensure Certbot runs:

```
sudo ln -s /opt/certbot/bin/certbot /usr/bin/certbot
```

## Install Certbot in Ubuntu with snapd

> Install snapd:

```
sudo apt install snapd
```

> Ensure you have the latest snapd version installed:

```
sudo snap install core; sudo snap refresh core
```

> Install Certbot with snapd:

```
sudo snap install --classic certbot
```

> Create a symlink to ensure Certbot runs:

```
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

## Create an SSL Certificate with Certbot

### Choose the best option for your needs

> Create SSL certs for all domains and configure redirects in the web server:

```
sudo certbot --apache

sudo certbot --nginx
```

> Create SSL certs for a specified domain (recommended if you’re using your system hostname):

```
sudo certbot --apache -d [domain] -d www.[domain]

sudo certbot --nginx -d [domain] -d www.[domain]
```

### If prompted, choose whether to redirect HTTP traffic to HTTPS – 1 (no redirect, no further changes to the server) or 2 (redirect all HTTP requests to HTTPS)

# Install Composer on Ubuntu

## To quickly install Composer globaly, run the following script in your terminal

```
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === '55ce33d7678c5a611085589f1f3ddf8b3c52d662cd01d4ba75c0ee0459970c2200a51f492d557530c71c15d8dba01eae') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"

sudo mv composer.phar /usr/local/bin/composer
```

# Install Laravel 7 in current directory

### Make sure directory is empty

```
composer create-project --prefer-dist laravel/laravel:^7.0 .
```

### Generate a key

```
php artisan key:generate
```

### Larevel Installed successfully on your machine

# Install osiset shopify app dependencies in Laravel using composer

```
composer require osiset/laravel-shopify
```

## Configuration

### Package

```
php artisan vendor:publish --tag=shopify-config
```

You're now able to access config in config/shopify-app.php

Essentially you will need to fill in the app_name, api_key, api_secret, and api_scopes to generate a working app. Its recommended you use an env file for the configuration.

### Shopify App

In your app's settings on your Shopify Partner dashboard, you need to set the callback URL to be:

```
https://example.com/
```

And the redirect_uri to be:

```
https://example.com/authenticate
```

The callback URL will point to the home route, while the redirect_uri will point to the authentication route.

NOTE:Those two URLs must start with https, otherwise you will get an error message: "Oauth error invalid_request: The redirect_uri is not whitelisted"

### Routing

This package expects a route named home to exist. By default, the package has this route defined which shows a simple welcome page. To enable it, you will need to open routes/web.php and comment out the default Laravel route for /.

```
Route::get('/', function () {
    return view('welcome');
})->middleware(['verify.shopify'])->name('home');
```

Next, modify resources/views/welcome.blade.php to extend this packages' layout for Shopify AppBridge abilities, example:

```
@extends('shopify-app::layouts.default')

@section('content')
    <!-- You are: (shop domain name) -->
    <p>You are: {{ $shopDomain ?? Auth::user()->name }}</p>
@endsection

@section('scripts')
    @parent

    <script>
        actions.TitleBar.create(app, { title: 'Welcome' });
    </script>
@endsection
```

### Models

You will need to modify your Laravel user model. Normally located in app/User.php or app/Models/User.php.

Open the file, add after the namespace:

```
use Osiset\ShopifyApp\Contracts\ShopModel as IShopModel;
use Osiset\ShopifyApp\Traits\ShopModel;
```

Next, modify the class line to become:

```
class User extends Authenticatable implements IShopModel
```

Next, inside the class, add:

```
use ShopModel;
```

A completed User.php file will look like this:

```
<?php

namespace App;

use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Osiset\ShopifyApp\Contracts\ShopModel as IShopModel;
use Osiset\ShopifyApp\Traits\ShopModel;

class User extends Authenticatable implements IShopModel
{
    use Notifiable;
    use ShopModel;

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        'name', 'email', 'password',
    ];

    /**
     * The attributes that should be hidden for arrays.
     *
     * @var array
     */
    protected $hidden = [
        'password', 'remember_token',
    ];
}
```

### Jobs

#### ScriptTags Job

First, you need to get permission for write script tags to storefront. Add write_script_tags to api_scopes (or in SHOPIFY_API_SCOPES env variable) in config/shopify-app.php like the following:

```
'api_scopes' => env('SHOPIFY_API_SCOPES', 'read_products,write_products,write_script_tags'),
```

#### Uninstalled Job (recommended)

There is a default job provided which soft deletes the shop, and its charges (if any) for you. You're able to install this job directly or extend it.

To install, first run:
``` php artisan vendor:publish --tag=shopify-jobs ```  a job will be placed in ``` App/Jobs/AppUninstalledJob ```.
Next, edit ``` config/shopify-app.php ``` to enable the job:

```
    'webhooks' => [
        [
            'topic' => env('SHOPIFY_WEBHOOK_1_TOPIC', 'APP_UNINSTALLED'),
            'address' => env('SHOPIFY_WEBHOOK_1_ADDRESS', 'https://(your-domain).com/webhook/app-uninstalled')
        ],
    ],
```

### Migrations

#### Automatic
By default, running php artisan migrate is enough. If you'd like to edit or review the migrations before running migrate, see Manual below.

#### Manual
Set env SHOPIFY_MANUAL_MIGRATIONS=1 or edit config/shopify-app.php, then run:
```
php artisan vendor:publish --tag=shopify-migrations
```
This will publish the migrations to your app's migration folder. When you're satisfied, run php artisan migrate to commit the changes.

### CSRF
You must disable CSRF as there is currently no solution for verifying session tokens with CSRF, there is a conflict due to new login creation each request.

Open \App\Http\Middleware\VerifyCsrfToken.php, and add or edit:
```
protected $except = [
    '*',
];
```
This will exclude all routes from verifying CSRF.
