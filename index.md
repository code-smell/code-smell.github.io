---
layout: default
title: Template for Jenkins Jobs for PHP Projects
---

# Template for Jenkins Jobs for PHP Projects

Until today Jenkins is the leading open-source continuous integration server. There are many other CI servers around, but Jenkins supports building and testing virtually any project. Thanks to its thriving plugin ecosystem. So, why not use it for PHP projects?

The goal of this project is to provide a standard template for Jenkins jobs for PHP projects. It is heavily inspired by Sebastian Bergmann's "Jenkins PHP Template", but tries to go one step further.

Why is Continuous Integration for web projects so important? The answer is that web applications are changed constantly and quickly. Environment parameters like the size or the behaviour of the user base, is constantly changing. What is sufficient today can be insufficient tomorrow. Monitoring and continuous improving of the internal quality when developing and maintaining a software is crucial, especially in a web environment.

In this project i'm going to guide you through the installation and configuration process. Simply follow these steps to get started:

1. Install the required Jenkins plugins and PHP tools
2. Orchestrate the PHP tools using Apache Ant (or Phing)
3. Configure the PHP tools for use with Jenkins job template
4. Create a Jenkins job for your PHP project
5. Monitor and improve your web application
