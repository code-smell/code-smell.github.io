---
layout: post
title: PHP Continuous Delivery with Jenkins CI
---

Jenkins is the leading open-source Continuous Integration server but is it suitable for a Continuous Delivery workflow
too? In this article we are going to use Jenkins to implement a simple Continuous Delivery workflow for a PHP 
application using Phing and Rocketeer.

## Prerequisites
{:id="prerequisites"}

This article depends on a previous article [Deploying a PHP Application with Rocketeer](2016/10/08/deploying-php-application-with-rocketeer.html).
Before you walk through this article you **must** have read the depending article.

### Phing

Install Phing globally.

## Install Jenkins plugins
{:id="install"}

- [Phing][jkPhing]{:target="_blank"} is an alternative to Apache Ant and allows you to use [Phing][phing]{:target="_blank"} to build PHP projects 

You can install the above plugins by using the web frontend or [Jenkins CLI][jkCLI]{:target="_blank"} client.

## Setup a PHP project with a deploying/delivery workflow using Rocketeer
{:id="automated-build"}

Follow the steps described in [Deploying a PHP Application with Rocketeer](2016/10/08/deploying-php-application-with-rocketeer.html)
to setup a sample PHP project.

## Setup Jenkins Job

1. Click on "New Item", choose "Freestyle project" and enter an item name.
2. Fill in your "Source Code Management" information.
3. Configure a "Build Trigger", for instance "Poll SCM".
4. Add a build step and choose "Invoke Phing targets"
   - select a Phing Version
   - fill in the Target "deploy"
   - in the Properties field add  
     host=your_host  
     username=your_username  
     password=your_password    
5. Click "Save".

To test the Jenkins Job click on the job name, and click on "Build Now".

## Real-world scenario

In a real-world scenario you probably would configure another "Build Trigger". If your project should be build after 
other projects are built you can check the option "Build after other projects are built" and fill in a "Project to 
watch". 

With such a configuration it is possible to chain certain Jenkins Jobs:

- Execute Job "Continuous Integration" 
- If it finishes successfully execute Job "Continuous Delivery"
- If it finishes successfully execute Job "XYZ"
- etc.



[jkPhing]: https://wiki.jenkins-ci.org/display/JENKINS/Phing+Plugin
[phing]: https://www.phing.info/trac/
[jkCLI]: https://wiki.jenkins-ci.org/display/JENKINS/Jenkins+CLI
