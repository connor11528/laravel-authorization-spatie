Laravel Subscription Billing and User Management 
===

layout: post
title: User Authorization and Authentication using Laravel 5
date: 2017-08-22
category: Laravel
lede: "In this post we're going to use the Laravel-Permission package by the infamous spatie web development agency to manage user access and permissions. Users can login to their dashboards where an admin can login to see root level access information."
author: Connor Leech
published: true
---

For this tutorial we are going to use the [spatie/laravel-permission](https://github.com/spatie/laravel-permission) package. 

Mad props to Caleb Oki from Nigeria for first outlining these steps in his post on [scotch.io](https://scotch.io/tutorials/user-authorization-in-laravel-54-with-spatie-laravel-permission)

### Create the application 

You must have all of the required environments installed on your machine. A lot of people recommend setting valet or vagrant for local Laravel development. I have not found it necessary. I also to find it overly complex for beginners. If you are new to the framework, check out [this in depth post](http://connorleech.info/blog/Build-an-online-forum-with-Laravel%E2%80%8A-Initial-Setup-and-Seeding-Part-1/) for creating the Laravel app and connecting it to a MySQL database on your local machine. I honestly refer back to it every time I create a new Laravel app because I forget the `mysql -uroot -p` command (maybe now that I've written it again I will remember).

Apart from creating a database, one command will generate our functional project code:

```
$ laravel new laravel-subscription-billing
```

Additionally, we need to generate keys and do other housekeeping when we create a new project. I created a gist [available here](https://gist.github.com/connor11528/fcfbdb63bc9633a54f40f0a66e3d3f2e) that allows you to do this all in one step.

Save the file and run it as `$ ./create_laravel_app.sh NAME_OF_YOUR_APP`

This will run a shell script to set up your application. The full contents of this shell script are featured below.

```
#!/bin/bash

laravel new $1
cd $1
composer install
yarn install 
touch README.md   
cp .env.example .env
git init
git add -A
git commit -m 'Initial commit'
php artisan key:generate
php artisan config:clear  
php artisan config:cache 
```

As you can see the script takes care of installing dependencies, copying our environment file, adding git and crafting application keys.

Eventually we're going to add subscription payment processing with Laravel Cashier and Stripe but first we're building out the user management and access control system.

### Add Laravel Permission

We can pull the laravel-permission package into our project by using Composer. 

Run this command to add the spatie/laravel-permission package:

```
$ composer require spatie/laravel-permission
```

This command adds the package to our **composer.json** file. Whenever anyone else uses our code and installs dependencies the spatie/laravel-permission package will also be installed.

Now that we've installed a new composer package we must let the rest of our Laravel application know about it. This happens within the **config/app.php** file and the `'providers'` array. [Laravel service providers](https://laravel.com/docs/5.4/providers) are the central mechanism where Laravel bootstraps our application and registers components. 

Add the laravel-permission package to our providers bootstrap script:

```
'providers' => [
    ...
    Spatie\Permission\PermissionServiceProvider::class,
]
```

You can start your app using `php artisan serve`. When we are compiling frontend code with Node.js moving forward we'll also need a separate terminal window running with `yarn run watch`. This will give us live code reloading in the browser and totally be worth the extra terminal window.

### Database Migrations

So we have created our app and added the package to our bootstrap script in **config/app.php**. The next thing to take note of is that the Laravel Permission would like to add some helper tables for our application. There is a migration file [available here](https://github.com/spatie/laravel-permission/blob/master/database/migrations/create_permission_tables.php.stub) that outlines exactly what they'd like to create. The table names are configurable, but by default the Laravel Permission package wants to create tables with the following functionality:

- Permissions
- Roles
- Model Has Permissions
- Model Has Roles
- Role Has Permissions 

Run this migration file with the following command:

```
$ php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider" --tag="migrations"
$ php artisan migrate
```

This sets up the Users, Roles and Permissions database models and their associated relationships.

Publish the [configuration file](https://github.com/spatie/laravel-permission/blob/master/config/permission.php) required for this package by running:

```
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider" --tag="config"
```

### Extending our User model 

Now that our database is set up to use the package we can modify our user model to take advantage of these authorization features.

Head into **app/User.php** and add a trait to the model called `HasRoles`. Traits are for extending classes. If a class uses a Trait then we can use the trait's methods in instances of the class. If you are new to OOP I suggest reading up about inheritance, traits and classes. To keep going though all we need to add is this:

```
class User extends Authenticatable
{
    use Notifiable;
    use HasRoles;
    ...
```

### Creating Roles and Permissions 

Code that we'll use in our controllers.
```
use Spatie\Permission\Models\Role;
use Spatie\Permission\Models\Permission;

$role = Role::create(['name' => 'writer']);
$permission = Permission::create(['name' => 'edit articles']);
```

To get the permissions for a user: `$permissions = $user->permissions;`

Get role name for user: `$roles = $user->roles()->pluck('name');`

Other mehtods, as outlined in [the documentation](https://github.com/spatie/laravel-permission/blob/master/README.md)

![](http://i.imgur.com/V1GS8wS.png)

### Generate the Authentication scaffolding

```
$ php artisan make:auth 
$ php artisan migrate
```

We're also going to change **app/Http/RegisterController.php** so that 



