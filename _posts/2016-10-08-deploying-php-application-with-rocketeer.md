---
layout: post
title: Deploying a PHP Application with Rocketeer
---

Rocketeer is a fast and easy deploying tool for PHP developers. It is inspired by Capistrano which is written in Ruby, but can easily be used to deploy any language. Since Rocketeer is written in PHP it has a few advantages over Capistrano especially in a PHP environment.

Rocketeer requires an SSH connection, meaning if your project is running on a shared hosting (and use FTP to upload files), it probably can't do anything for you.

Rocketeer provides a handful of built-in tasks to deploy and manage your remote projects.

## Core folders

Two configuration options are essential when deploying with Rocketeer.

The `root_directory` on your server – this folder is Rocketeer's little self-contained world: whatever it does will be in that folder. 
And the `application_name` - this is to let Rocketeer handle multiple applications on the same server.

If per example your `root_directory` is `/var/www/` and your `application_name` is `webapp`, then Rocketeer will create `/var/www/webapp/` and everything it does will happen in that folder.

## How it works

In the working folder mentioned above on the server Rocketeer will create three folders.

- `releases` is where the history of your application is stored. 
- `current` is where the **latest** version of your application will always be. This is a **symlink** pointing to one of the releases in the `releases` folder.
- `shared` is where files are stored, that are shared between each releases. 

When a new release is created and is ready to be served, Rocketeer will update the symlink of the current folder to make it point to the new release.

	| var
	|-- www
	  |-- webapp
	    |-- current => /var/www/webapp/releases/20130721000000
	    |-- releases
	    |  |-- 20130721000000
	    |  |  |-- app
	    |  |     |-- storage
	    |  |       |-- logs => /var/www/webapp/shared/app/storage/logs
	    |  | 20130602000000
	    |-- shared
	      |-- app
	        |-- storage
	          |-- logs

## Template for PHP application with Rocketeer

I have prepared a template for a simple PHP application and the deployment with Rocketeer. 
In order to work you must have Git installed on your machine.

### 1. Clone repository 

    $ git clone https://github.com/code-smell/php-deploy-rocketeer.git
        
### 2. Install Rocketeer
        
    $ cd php-deploy-rocketeer
    $ php composer.phar install            
    
### 3. Edit configuration

In file `.rocketeer/remote.php` edit the following line:
    
    'root_directory' => 'your_root_directory',

In file `.rocketeer/config.php` edit the following lines:

    'host'      => 'your_host',
    'username'  => 'your_username',
    'password'  => 'your_password',
    
If you leave the entries above empty, you will be asked by Rocketeer.
     
Please note that the folder `.rocketeer` is hidden and therefore is not visible in your file explorer.
    
### 4. Deploy application
    
Now, you are ready to deploy. 
    
    $ cd php-deploy-rocketeer
    $ php vendor/bin/rocketeer deploy
    
You can skip the questions about the repository username and password asked by Rocketeer. After finishing you can 
display the current release. 

    $ php vendor/bin/rocketeer current
    
To list the available commands use the following command.

    $ php vendor/bin/rocketeer list
    
### 5. Open website

Depending on your remote server configuration you can access the PHP application by
typing: 

    http://your-webserver.test/

The output should be something like this:

    Rocketeer Release 20161015204946.        
    
## What's next

Of course, this was only a simple example. There isn't a database nor a complex structure in this
application. But it clearly shows that deploying with Rocketeer is quite easy. Now it's up to you.
