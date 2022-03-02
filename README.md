
# Introduction
Over a decade Laravel has evolved and become the most widely used PHP framework to create projects from small to large projects that require better performance.

In terms of designing and building an application, deployment is one of the important things that must be considered. In this tutorial I want to take Laravel to a more advanced level, we will learn how to make zero downtime deployment with laravel and github actions.

## Content

 1. What is CI/CD or Zero downtime deployment
 2. Why using Github Actions
 3. Preparations
 4. Install and Setup Deployer on Project
 5. Prepare the Server
 6. Setup Github Action
 7. Conclusion

### 1. What is CI/CD or Zero downtime deployment
CI and CD stand for continuous integration and continuous delivery/continuous deployment. CI/CD allows organizations to ship software quickly and efficiently. CI/CD facilitates an effective process for getting products to market faster than ever before, continuously delivering code into production, and ensuring an ongoing flow of new features and bug fixes via the most efficient delivery method.

Zero downtime deployment is **a deployment method where your website or application is never down or in an unstable state during the deployment process**. To achieve this the web server doesn't start serving the changed code until the entire deployment process is complete

###  2. Why using Github Actions
in this particular case I teach you how to implement Zero Downtime Deployment using a **[GitHub Actions](https://github.com/features/actions)** and **deployment tool** called **[Deployer](https://deployer.org/#what-is-deployer)**. I will not explain the function of the two in details, but here is some reason why i choose this:
- Github actions [offer](https://docs.github.com/en/billing/managing-billing-for-github-actions/about-billing-for-github-actions) free 2000 minutes per month for both private and public repo meanwhile others only offer free on public repo only.
- This offer can last on small and middle projects that requires 15-30 minutes each run

### 3. Preparations

 - [ ] Your Laravel Project
 - [ ] Hosting that can access through ssh (e.g: [DigitalOcean](https://www.digitalocean.com/)
 - [ ] Domain Name
 
##  Install and Setup Deployer on Project

### 1. Install Deployer

First, open your Laravel Project and run following command:

    composer require --dev deployer/dist

After installation complete, run:

    dep init

Deployer will ask you a few question and after finishing you will have a **deploy.php** or **deploy.yaml** file but we'll continue with **deploy.php**. This is our deployment recipe. It contains hosts, tasks and requires other recipes. All framework recipes that come with Deployer are based on the [common](https://deployer.org/docs/7.x/recipe/common) recipe.

In this particular article you can copy and paste below config on your deploy.php

### 2. Copy the configuration
     <?php
    
    namespace Deployer;
    
    require 'recipe/laravel.php';
    require 'contrib/npm.php';
    require 'contrib/rsync.php';
    
    set('application', 'Your Project Name');
    set('repository', 'git@github.com:change-with-your-repo-url.git'); // Git Repository
    set('ssh_multiplexing', true);  // Speed up deployment
    //set('default_timeout', 1000);
    
    set('rsync_src', function () {
        return __DIR__; // If your project isn't in the root, you'll need to change this.
    });
    
    // Configuring the rsync exclusions.
    // You'll want to exclude anything that you don't want on the production server.
    add('rsync', [
        'exclude' => [
            '.git',
            '/vendor/',
            '/node_modules/',
            '.github',
            'deploy.php',
        ],
    ]);
    
    // Set up a deployer task to copy secrets to the server.
    // Grabs the dotenv file from the github secret
    task('deploy:secrets', function () {
        file_put_contents(__DIR__ . '/.env', getenv('DOT_ENV'));
        upload('.env', get('deploy_path') . '/shared');
    });
    
    // Hosts
    host('prod') // Name of the server
    ->setHostname('xxx.xxx.xxx.xxx') // Hostname or IP address
    ->set('remote_user', 'root') // SSH user
    ->set('branch', 'main') // Git branch
    ->set('deploy_path', '/var/www/project-path'); // Deploy path
    
    after('deploy:failed', 'deploy:unlock');  // Unlock after failed deploy
    
    desc('Start of Deploy the application');
    
    task('deploy', [
        'deploy:prepare',
        'rsync',                // Deploy code & built assets
        'deploy:secrets',       // Deploy secrets
        'deploy:vendors',
        'deploy:shared',        // 
        'artisan:storage:link', //
        'artisan:view:cache',   //
        'artisan:config:cache', // Laravel specific steps
        'artisan:migrate',      //
        'artisan:queue:restart',//
        'deploy:publish',       // 
    ]);
   
    desc('End of Deploy the application');

The code above is self explanatory, so you can read it.


### Add Github Workflow

Create new folder on project root 

    .github/workflows/push_workflow.yml

Then copy below code 

    name: CI/CD Workflow
    
    on:
      push:
        branches:
          - main
    
    jobs:
      build-js-production:
        name: Build JavaScript/CSS for Production Server
        runs-on: ubuntu-latest
        if: github.ref == 'refs/heads/main'
        steps:
          - uses: actions/checkout@v1
          - name: NPM Build
            run: |
              npm install
              npm run prod
          - name: Put built assets in Artifacts
            uses: actions/upload-artifact@v1
            with:
              name: assets
              path: public
              retention-days: 3
      deploy-production:
        name: Deploy Project to Production Server
        runs-on: ubuntu-latest
        needs: [ build-js-production ]
        if: github.ref == 'refs/heads/main'
        steps:
          - uses: actions/checkout@v1
          - name: Fetch built assets from Artifacts
            uses: actions/download-artifact@v1
            with:
              name: assets
              path: public
          - name: Setup PHP
            uses: shivammathur/setup-php@master
            with:
              php-version: '8.0'
              extension-csv: mbstring, bcmath
          - name: Composer install
            run: composer install -q --no-ansi --no-interaction --no-scripts --no-progress --prefer-dist
          - name: Setup Deployer
            uses: atymic/deployer-php-action@master
            with:
              ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
              ssh-known-hosts: ${{ secrets.SSH_KNOWN_HOSTS }}
          - name: Deploy to Production
            env:
              DOT_ENV: ${{ secrets.DOT_ENV_PRODUCTION }}
            run: php vendor/bin/dep deploy prod --branch="main" -vv

**Explanation:** 
- If any push on ***"main"*** branch
- It will run jobs ***"build-js-production"*** that will build and compile our front end using command ***"npm install"*** & ***"npm run prod"*** then upload the compiling assets to in Artifacts
- Next, after the second step finish, it will run the second job which is ***"deploy-production"*** , that will deploy our code to the server and trigger the ***"deploy.php"***  that we already prepare.

it's not finish, we still need to fill credentials about our server so it can access the server, the credentials we looking is below
- secrets.SSH_PRIVATE_KEY
- secrets.SSH_KNOWN_HOSTS
- secrets.DOT_ENV_PRODUCTION

### Add Credentials to Github Secrets 
1. SSH Private Key
2. SSH Known Hosts
3. DOT_ENV_Production


## Conclussion
So let me recap what we've done, so after doing all the step above our code will automatically publish to server and go live. You can also seperate the server with "Staging" & Production" and also run your own test right on this workflow.
