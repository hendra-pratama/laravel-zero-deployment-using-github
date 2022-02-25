# Introduction
Over a decade Laravel has evolved and become the most widely used PHP framework to create projects from small to large projects that require better performance.

In terms of designing and building an application, deployment is one of the important things that must be considered. In this tutorial I want to take Laravel to a more advanced level, we will learn how to make zero downtime deployment with laravel and github actions.

## Content

 1. What is CI/CD or Zero downtime deployment
 2. The Strategy
 4. Install and Setup Deployer on Project
 5. Prepare the Server
 6. Setup Github Action
 7. Conclusion

### 1. What is CI/CD or Zero downtime deployment
CI and CD stand for continuous integration and continuous delivery/continuous deployment. CI/CD allows organizations to ship software quickly and efficiently. CI/CD facilitates an effective process for getting products to market faster than ever before, continuously delivering code into production, and ensuring an ongoing flow of new features and bug fixes via the most efficient delivery method.

Zero downtime deployment is **a deployment method where your website or application is never down or in an unstable state during the deployment process**. To achieve this the web server doesn't start serving the changed code until the entire deployment process is complete

###  2. The Strategy
in this particular case I teach you how to implement Zero Downtime Deployment using a **[GitHub Actions](https://github.com/features/actions)** and **deployment tool** called **[Deployer](https://deployer.org/#what-is-deployer)**. I will not explain the function of the two in details, in a essence

##  Install and Setup Deployer on Project

### 1. Install Deployer

First, open your Laravel Project and run following command:

    composer require --dev deployer/dist

After installation complete, run:

    dep init

Deployer will ask you a few question and after finishing you will have a **deploy.php** or **deploy.yaml** file but we'll continue with **deploy.php**. This is our deployment recipe. It contains hosts, tasks and requires other recipes. All framework recipes that come with Deployer are based on the [common](https://deployer.org/docs/7.x/recipe/common) recipe.

### Config our Task
### 
--
--
--


## Conclussion
