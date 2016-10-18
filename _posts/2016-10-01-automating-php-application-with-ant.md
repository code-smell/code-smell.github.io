---
layout: post
title: Automating build process of a PHP Application with Apache Ant
---

Apache Ant is a software tool for automating software build processes. It is implemented using the Java language, 
requires the Java platform, and is best suited to building Java projects. In this article we are using Ant to automate the build process of a PHP application. 

Please refer to the Git repository on [GitHub][repository]{:target="_blank"}.

## Requirements

In order to work the following requirements have to be installed properly.

- [Git][git]{:target="_blank"} - a distributed version control system 
- [Apache Ant][ant]{:target="_blank"} - a command-line build tool
- [Composer][composer]{:target="_blank"} - Dependency Manager for PHP
                                           
Please refer to the appropriate documention which guides you through the installation process.

## The workflow

A simplified PHP build process workflow follows looks like this.

<img class="pure-img" src="/img/php-workflow-with-ant.png" alt="a simplified PHP build process">

1. The build process is invoked manually on the command line CLI for example.
2. Apache Ant first downloads the Composer PHP-Archive and starts the install process of Composer.
3. Composer installs all PHP-Libraries that are listed in the config file like PHPUnit, PHPCS, Pdepend or PhpDox.
4. Apache Ant loops through the PHP-Tools and executes them, writes Logfiles and prints out human readable messages to the CLI.

## Git

Use the following commands to clone the project to your local machine and change to the project directory. 

~~~bash
$ git clone https://github.com/code-smell/php-automate-build-ant.git my-project
$ cd my-project
~~~
   
## Apache Ant - Build Script
    
The project is set up and ready for Ant to automate. Ant orchestrates the execution of several software quality and 
metric tools defined in the `build.xml` build script like:

- Download Composer PHP Archive and install PHP dependencies using Composer 
- Perform syntax check of PHP sourcecode files using php's lint-mode
- Measure project size using PHPLOC
- Calculate software metrics using PHP_Depend
- Perform project mess detection using PHPMD
- Find coding standard violations using PHP_CodeSniffer
- Find duplicate code using PHPCPD
- Run unit tests with PHPUnit
- Generate project documentation using phpDox
- Log results in several formats for later use in a Continuous Integration environment 

The `build.xml` script is stored in the root folder.

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<project name="php-automate-build-ant" default="full-build">

    <!-- By default the tools are managed by Composer in ${basedir}/vendor/bin -->
    <property name="pdepend" value="${basedir}/vendor/bin/pdepend"/>
    <property name="phpcpd" value="${basedir}/vendor/bin/phpcpd"/>
    <property name="phpcs" value="${basedir}/vendor/bin/phpcs"/>
    <property name="phpdox" value="${basedir}/vendor/bin/phpdox"/>
    <property name="phploc" value="${basedir}/vendor/bin/phploc"/>
    <property name="phpmd" value="${basedir}/vendor/bin/phpmd"/>
    <property name="phpunit" value="${basedir}/vendor/bin/phpunit"/>

    <target name="clean" unless="clean.done" description="Cleanup build artifacts.">
        <delete dir="${basedir}/build/api"/>
        <delete dir="${basedir}/build/coverage"/>
        <delete dir="${basedir}/build/logs"/>
        <delete dir="${basedir}/build/pdepend"/>
        <delete dir="${basedir}/build/phpdox"/>
        <property name="clean.done" value="true"/>
    </target>

    <target name="composer" description="Install composer packages including require-dev.">
        <get src="https://getcomposer.org/download/1.2.1/composer.phar" dest="composer.phar"/>
        <exec executable="php" failonerror="true">
            <arg value="${basedir}/composer.phar"/>
            <arg value="install"/>
            <arg value="--prefer-dist"/>
            <arg value="--no-progress"/>
        </exec>
    </target>

    <target name="full-build" depends="prepare,composer,static-analysis,phpunit,phpdox,-check-failure"
            description="Perform static analysis, run tests, and generate project documentation.">
        <echo message="Built"/>
    </target>

    <target name="lint" unless="lint.done" description="Perform syntax check of PHP sourcecode files.">
        <apply executable="php" failonerror="true" taskname="lint">
            <arg value="-l"/>
            <fileset dir="${basedir}/src">
                <include name="**/*.php"/>
                <!-- modified/ -->
            </fileset>
            <fileset dir="${basedir}/tests">
                <include name="**/*.php"/>
                <!-- modified/ -->
            </fileset>
        </apply>
        <property name="lint.done" value="true"/>
    </target>

    <target name="pdepend" unless="pdepend.done" depends="prepare"
            description="Calculate software metrics using PHP_Depend and log result in XML format. Intended for usage within a continuous integration environment.">
        <exec executable="${pdepend}" taskname="pdepend">
            <arg value="--jdepend-xml=${basedir}/build/logs/jdepend.xml"/>
            <arg value="--jdepend-chart=${basedir}/build/pdepend/dependencies.svg"/>
            <arg value="--overview-pyramid=${basedir}/build/pdepend/overview-pyramid.svg"/>
            <arg path="${basedir}/src"/>
        </exec>
        <property name="pdepend.done" value="true"/>
    </target>

    <target name="phpcpd" unless="phpcpd.done"
            description="Find duplicate code using PHPCPD and print human readable output. Intended for usage on the command line before committing.">
        <exec executable="${phpcpd}" taskname="phpcpd">
            <arg path="${basedir}/src"/>
        </exec>
        <property name="phpcpd.done" value="true"/>
    </target>

    <target name="phpcpd-ci" unless="phpcpd.done" depends="prepare"
            description="Find duplicate code using PHPCPD and log result in XML format. Intended for usage within a continuous integration environment.">
        <exec executable="${phpcpd}" taskname="phpcpd">
            <arg value="--log-pmd"/>
            <arg path="${basedir}/build/logs/pmd-cpd.xml"/>
            <arg path="${basedir}/src"/>
        </exec>
        <property name="phpcpd.done" value="true"/>
    </target>

    <target name="phpcs" unless="phpcs.done"
            description="Find coding standard violations using PHP_CodeSniffer and print human readable output. Intended for usage on the command line before committing.">
        <exec executable="${phpcs}" taskname="phpcs">
            <arg value="--standard=PSR2"/>
            <arg value="--extensions=php"/>
            <arg value="--ignore=autoload.php"/>
            <arg path="${basedir}/src"/>
            <arg path="${basedir}/tests"/>
        </exec>
        <property name="phpcs.done" value="true"/>
    </target>

    <target name="phpcs-ci" unless="phpcs.done" depends="prepare"
            description="Find coding standard violations using PHP_CodeSniffer and log result in XML format. Intended for usage within a continuous integration environment.">
        <exec executable="${phpcs}" output="/dev/null" taskname="phpcs">
            <arg value="--report=checkstyle"/>
            <arg value="--report-file=${basedir}/build/logs/checkstyle.xml"/>
            <arg value="--standard=PSR2"/>
            <arg value="--extensions=php"/>
            <arg value="--ignore=autoload.php"/>
            <arg path="${basedir}/src"/>
            <arg path="${basedir}/tests"/>
        </exec>
        <property name="phpcs.done" value="true"/>
    </target>

    <target name="phpdox" unless="phpdox.done" depends="phploc-ci,phpcs-ci,phpmd-ci"
            description="Generate project documentation using phpDox.">
        <exec executable="${phpdox}" dir="${basedir}/build" taskname="phpdox"/>
        <property name="phpdox.done" value="true"/>
    </target>

    <target name="phploc" unless="phploc.done"
            description="Measure project size using PHPLOC and print human readable output. Intended for usage on the command line.">
        <exec executable="${phploc}" taskname="phploc">
            <arg value="--count-tests"/>
            <arg path="${basedir}/src"/>
            <arg path="${basedir}/tests"/>
        </exec>
        <property name="phploc.done" value="true"/>
    </target>

    <target name="phploc-ci" unless="phploc.done" depends="prepare"
            description="Measure project size using PHPLOC and log result in CSV and XML format. Intended for usage within a continuous integration environment.">
        <exec executable="${phploc}" taskname="phploc">
            <arg value="--count-tests"/>
            <arg value="--log-csv"/>
            <arg path="${basedir}/build/logs/phploc.csv"/>
            <arg value="--log-xml"/>
            <arg path="${basedir}/build/logs/phploc.xml"/>
            <arg path="${basedir}/src"/>
            <arg path="${basedir}/tests"/>
        </exec>
        <property name="phploc.done" value="true"/>
    </target>

    <target name="phpmd" unless="phpmd.done"
            description="Perform project mess detection using PHPMD and print human readable output. Intended for usage on the command line before committing.">
        <exec executable="${phpmd}" taskname="phpmd">
            <arg path="${basedir}/src"/>
            <arg value="text"/>
            <arg path="${basedir}/build/phpmd.xml"/>
        </exec>
        <property name="phpmd.done" value="true"/>
    </target>

    <target name="phpmd-ci" unless="phpmd.done" depends="prepare"
            description="Perform project mess detection using PHPMD and log result in XML format. Intended for usage within a continuous integration environment.">
        <exec executable="${phpmd}" taskname="phpmd">
            <arg path="${basedir}/src"/>
            <arg value="xml"/>
            <arg path="${basedir}/build/phpmd.xml"/>
            <arg value="--reportfile"/>
            <arg path="${basedir}/build/logs/pmd.xml"/>
        </exec>
        <property name="phpmd.done" value="true"/>
    </target>

    <target name="phpunit" unless="phpunit.done" depends="prepare" description="Run unit tests with PHPUnit.">
        <exec executable="${phpunit}" resultproperty="result.phpunit" taskname="phpunit">
            <arg value="--configuration"/>
            <arg path="${basedir}/build/phpunit.xml"/>
            <arg path="tests"/>
        </exec>
        <property name="phpunit.done" value="true"/>
    </target>

    <target name="phpunit-no-coverage" unless="phpunit.done" depends="prepare"
            description="Run unit tests with PHPUnit without generating code coverage reports.">
        <exec executable="${phpunit}" failonerror="true" taskname="phpunit">
            <arg value="--configuration"/>
            <arg path="${basedir}/build/phpunit.xml"/>
            <arg path="tests"/>
            <arg value="--no-coverage"/>
        </exec>
        <property name="phpunit.done" value="true"/>
    </target>

    <target name="prepare" unless="prepare.done" depends="clean" description="Prepare for build.">
        <mkdir dir="${basedir}/build/api"/>
        <mkdir dir="${basedir}/build/coverage"/>
        <mkdir dir="${basedir}/build/logs"/>
        <mkdir dir="${basedir}/build/pdepend"/>
        <mkdir dir="${basedir}/build/phpdox"/>
        <property name="prepare.done" value="true"/>
    </target>

    <target name="quick-build" depends="prepare,composer,lint,phpunit-no-coverage"
            description="Perform lint check and run tests without generating code coverage reports.">
        <echo message="Built"/>
    </target>

    <target name="static-analysis" depends="lint,phploc-ci,pdepend,phpmd-ci,phpcs-ci,phpcpd-ci"
            description="Perform static analysis.">
        <echo message="Done"/>
    </target>

    <target name="-check-failure">
        <fail message="PHPUnit did not finish successfully">
            <condition>
                <not>
                    <equals arg1="${result.phpunit}" arg2="0"/>
                </not>
            </condition>
        </fail>
        <echo message="Checked failure"/>
    </target>

</project>
~~~

## Composer

Composer is the package manager for PHP and plays a major role in this game. It installs all PHP dependencies defined 
in `composer.json`. The `build.xml` script includes a target named by `composer` which is invoked first when initiating 
one of the main targets (see below).

You can invoke the `composer` target manually to download Composer as a PHP Archive and install all PHP dependencies.

~~~bash
$ ant composer
~~~

The following PHP tools are available now. The related binaries are stored under `vendor/bin`.
    
- [PHP_CodeSniffer][PhpCodeSniffer]{:target="_blank"} tokenizes PHP, JavaScript and CSS files and detects violations of a defined set of coding standards.
- [PHPUnit][PhpUnit]{:target="_blank"} is a programmer-oriented testing framework for PHP. It is an instance of the xUnit architecture for unit testing frameworks.
- [PHP Copy/Paste Detector][PhpCpd]{:target="_blank"} is a Copy/Paste Detector (CPD) for PHP code.
- [phpDox][PhpDox]{:target="_blank"} - PHP Documentation Generator is a documentation generator for PHP projects. This includes, but is not limited to, API documentation.
- [PhpDepend][PhpDepend]{:target="_blank"} can generate a large set of software metrics from a given code base. These values can be used to measure the quality of a software project and to identify the parts of an application where a refactoring should be applied.
- [PHPLoc][PhpLoc]{:target="_blank"} - PHPLOC is a tool for quickly measuring the size of a PHP project.
- [PHPMD][PhpMd]{:target="_blank"} - PHP Mess Detector is a source code analyzer.
 
## Configure PHP tools

The build script assumes that the following sets are located in the `build` folder. Of course we could put everything into one file, but by dividing the configuration we make the files more readable by human.

### PHPUnit

The `phpunit` task in the `build.xml` assumes that an XML configuration file for PHPUnit is used to configure the following logging targets:

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit bootstrap="../tests/bootstrap.php"
         backupGlobals="false"
         backupStaticAttributes="false"
         strict="true"
         verbose="true">
    <!--
    <testsuites>
        <testsuite name="ProjectName">
            <directory suffix="Test.php">tests/unit/</directory>
            <directory suffix="Test.php">tests/integration/</directory>
        </testsuite>
    </testsuites>
    -->
    <logging>
        <log type="coverage-html" target="coverage"/>
        <log type="coverage-clover" target="logs/clover.xml"/>
        <log type="coverage-crap4j" target="logs/crap4j.xml"/>
        <log type="junit" target="logs/junit.xml" logIncompleteSkipped="false"/>
    </logging>
    <filter>
        <whitelist processUncoveredFilesFromWhitelist="true">
            <directory suffix=".php">../src</directory>
        </whitelist>
    </filter>
</phpunit>
~~~

You can find more information in the [documentation for PHPUnit](https://phpunit.de/manual/current/en/appendixes.configuration.html){:target="_blank"}.

### phpDox

The `phpdox` task in the `build.xml` assumes that an XML configuration file for phpDox is used to configure the API documentation generation:

~~~xml
<phpdox xmlns="http://xml.phpdox.net/config">
 <project name="name-of-project" source="../src" workdir="../build/phpdox">
  <collector publiconly="false">
   <include mask="*.php" />
  </collector>
  <generator output="../build">
   <build engine="html" enabled="true" output="api">
    <file extension="html" />
   </build>
  </generator>
 </project>
</phpdox>
~~~

You can find more information in the [documentation for phpDox](http://phpdox.de/getting-started.html){:target="_blank"}.

### PHP_CodeSniffer

The `phpcs` and `phpcs-ci` tasks in the `build.xml` assume that an XML configuration file for PHP_CodeSniffer is used to configure the coding standard:

~~~xml
<ruleset name="name-of-your-coding-standard">
    <description>Description of your coding standard</description>
    <rule ref="Generic.PHP.DisallowShortOpenTag"/>
    <!-- ... -->
</ruleset>
~~~

The build script assumes that the rule sets for PHP_CodeSniffer is located at `build/phpcs.xml`.

You can find more information in the [documentation for PHP_CodeSniffer](https://github.com/squizlabs/PHP_CodeSniffer/wiki){:target="_blank"}.

### PHPMD

The `phpmd` and `phpmd-ci` tasks in the `build.xml` assume that an XML configuration file for PHPMD is used to configure the coding standard:

~~~xml
<?xml version="1.0"?>
<ruleset name="PHPMD rule sets"
         xmlns="http://pmd.sf.net/ruleset/1.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://pmd.sf.net/ruleset/1.0.0
                      http://pmd.sf.net/ruleset_xml_schema.xsd"
         xsi:noNamespaceSchemaLocation="http://pmd.sf.net/ruleset_xml_schema.xsd">
    <description>Custom rule sets that checks your PHP code.</description>
    <rule ref="rulesets/unusedcode.xml"/>
    <rule ref="rulesets/codesize.xml/CyclomaticComplexity"/>
    <rule ref="rulesets/codesize.xml/ExcessiveParameterList"/>
</ruleset>
~~~

You can find more information in the [documentation for PHPMD](https://phpmd.org/documentation/index.html){:target="_blank"}.
    
## Ant Targets

Here is an overview of the targets defined in the above `build.xml` script that are intended to be directly invoked:

Main targets:

- `full-build` - Perform static analysis, run tests, and generate project documentation.
- `quick-build` - Perform lint check and run tests without generating code coverage reports.
- `static-analysis` - Perform static analysis.
- `composer` - Install composer packages including require-dev.

Sub targets:

- `clean` - Cleanup build artifacts.
- `lint` - Perform syntax check of PHP sourcecode files.
- `pdepend` - Calculate software metrics using PHP_Depend and log result in XML format. Intended for usage within a continuous integration environment.
- `phpcpd` - Find duplicate code using PHPCPD and print human readable output. Intended for usage on the command line before committing.
- `phpcpd-ci` - Find duplicate code using PHPCPD and log result in XML format. Intended for usage within a continuous integration environment.
- `phpcs` - Find coding standard violations using PHP_CodeSniffer and print human readable output. Intended for usage on the command line before committing.
- `phpcs-ci` - Find coding standard violations using PHP_CodeSniffer and log result in XML format. Intended for usage within a continuous integration environment.
- `phpdox` - Generate project documentation using phpDox.
- `phploc` - Measure project size using PHPLOC and print human readable output. Intended for usage on the command line.
- `phploc-ci` - Measure project size using PHPLOC and log result in CSV and XML format. Intended for usage within a continuous integration environment.
- `phpmd` - Perform project mess detection using PHPMD and print human readable output. Intended for usage on the command line before committing.
- `phpmd-ci` - Perform project mess detection using PHPMD and log result in XML format. Intended for usage within a continuous integration environment.
- `phpunit` - Run unit tests with PHPUnit.
- `phpunit-no-coverage` - Run unit tests with PHPUnit without generating code coverage reports.
- `prepare` - Prepare for build.

### Full Build

The `full-build` target installs dependencies, performs static analysis, runs tests, and generates project documentation. 

~~~bash
$ ant full-build
~~~
    
#### Console Output
    
~~~bash    
Buildfile: /web/www/com.github/code-smell/php-automate-build-ant/build.xml

clean:
   [delete] Deleting directory /web/www/com.github/code-smell/php-automate-build-ant/build/api
   [delete] Deleting directory /web/www/com.github/code-smell/php-automate-build-ant/build/coverage
   [delete] Deleting directory /web/www/com.github/code-smell/php-automate-build-ant/build/logs
   [delete] Deleting directory /web/www/com.github/code-smell/php-automate-build-ant/build/pdepend
   [delete] Deleting directory /web/www/com.github/code-smell/php-automate-build-ant/build/phpdox

prepare:
    [mkdir] Created dir: /web/www/com.github/code-smell/php-automate-build-ant/build/api
    [mkdir] Created dir: /web/www/com.github/code-smell/php-automate-build-ant/build/coverage
    [mkdir] Created dir: /web/www/com.github/code-smell/php-automate-build-ant/build/logs
    [mkdir] Created dir: /web/www/com.github/code-smell/php-automate-build-ant/build/pdepend
    [mkdir] Created dir: /web/www/com.github/code-smell/php-automate-build-ant/build/phpdox

composer:
      [get] Getting: https://getcomposer.org/download/1.2.1/composer.phar
      [get] To: /web/www/com.github/code-smell/php-automate-build-ant/composer.phar
     [exec] You are running composer with xdebug enabled. This has a major impact on runtime performance. See https://getcomposer.org/xdebug
     [exec] Loading composer repositories with package information
     [exec] Installing dependencies (including require-dev) from lock file
     [exec] Nothing to install or update
     [exec] Generating autoload files

lint:
     [lint] No syntax errors detected in /web/www/com.github/code-smell/php-automate-build-ant/src/Bar.php
     [lint] No syntax errors detected in /web/www/com.github/code-smell/php-automate-build-ant/tests/BarTest.php
     [lint] No syntax errors detected in /web/www/com.github/code-smell/php-automate-build-ant/tests/bootstrap.php

phploc-ci:
   [phploc] phploc 3.0.1 by Sebastian Bergmann.
   [phploc] 
   [phploc] Directories                                          1
   [phploc] Files                                                3
   [phploc] 
   [phploc] Size
   [phploc]   Lines of Code (LOC)                               90
   [phploc]   Comment Lines of Code (CLOC)                      34 (37.78%)
   [phploc]   Non-Comment Lines of Code (NCLOC)                 56 (62.22%)
   [phploc]   Logical Lines of Code (LLOC)                      14 (15.56%)
   [phploc]     Classes                                          4 (28.57%)
   [phploc]       Average Class Length                           2
   [phploc]         Minimum Class Length                         0
   [phploc]         Maximum Class Length                         4
   [phploc]       Average Method Length                          0
   [phploc]         Minimum Method Length                        0
   [phploc]         Maximum Method Length                        1
   [phploc]     Functions                                        4 (28.57%)
   [phploc]       Average Function Length                        0
   [phploc]     Not in classes or functions                      6 (42.86%)
   [phploc] 
   [phploc] Cyclomatic Complexity
   [phploc]   Average Complexity per LLOC                     0.00
   [phploc]   Average Complexity per Class                    1.00
   [phploc]     Minimum Class Complexity                      1.00
   [phploc]     Maximum Class Complexity                      1.00
   [phploc]   Average Complexity per Method                   1.00
   [phploc]     Minimum Method Complexity                     1.00
   [phploc]     Maximum Method Complexity                     1.00
   [phploc] 
   [phploc] Dependencies
   [phploc]   Global Accesses                                    0
   [phploc]     Global Constants                                 0 (0.00%)
   [phploc]     Global Variables                                 0 (0.00%)
   [phploc]     Super-Global Variables                           0 (0.00%)
   [phploc]   Attribute Accesses                                 3
   [phploc]     Non-Static                                       3 (100.00%)
   [phploc]     Static                                           0 (0.00%)
   [phploc]   Method Calls                                       4
   [phploc]     Non-Static                                       4 (100.00%)
   [phploc]     Static                                           0 (0.00%)
   [phploc] 
   [phploc] Structure
   [phploc]   Namespaces                                         1
   [phploc]   Interfaces                                         0
   [phploc]   Traits                                             0
   [phploc]   Classes                                            1
   [phploc]     Abstract Classes                                 0 (0.00%)
   [phploc]     Concrete Classes                                 1 (100.00%)
   [phploc]   Methods                                            4
   [phploc]     Scope
   [phploc]       Non-Static Methods                             4 (100.00%)
   [phploc]       Static Methods                                 0 (0.00%)
   [phploc]     Visibility
   [phploc]       Public Methods                                 4 (100.00%)
   [phploc]       Non-Public Methods                             0 (0.00%)
   [phploc]   Functions                                          0
   [phploc]     Named Functions                                  0 (0.00%)
   [phploc]     Anonymous Functions                              0 (0.00%)
   [phploc]   Constants                                          0
   [phploc]     Global Constants                                 0 (0.00%)
   [phploc]     Class Constants                                  0 (0.00%)
   [phploc] 
   [phploc] Tests
   [phploc]   Classes                                            1
   [phploc]   Methods                                            2

pdepend:
  [pdepend] PDepend 2.2.4
  [pdepend] 
  [pdepend] Parsing source files:
  [pdepend] .     0
  [pdepend] 
  [pdepend] Calculating Dependency metrics:
  [pdepend] .     0
  [pdepend] 
  [pdepend] Calculating Coupling metrics:
  [pdepend] .     0
  [pdepend] 
  [pdepend] Calculating Cyclomatic Complexity metrics:
  [pdepend] .     0
  [pdepend] 
  [pdepend] Calculating Inheritance metrics:
  [pdepend] .     0
  [pdepend] 
  [pdepend] Calculating Node Count metrics:
  [pdepend] .     0
  [pdepend] 
  [pdepend] Calculating Node Loc metrics:
  [pdepend] .     0
  [pdepend] 
  [pdepend] Generating pdepend log files, this may take a moment.
  [pdepend] 
  [pdepend] Time: 0:00:00; Memory: 5.50Mb

phpmd-ci:

phpcs-ci:

phpcpd-ci:
   [phpcpd] phpcpd 2.0.4 by Sebastian Bergmann.
   [phpcpd] 
   [phpcpd] 0.00% duplicated lines out of 58 total lines of code.
   [phpcpd] 
   [phpcpd] Time: 61 ms, Memory: 2.75MB

static-analysis:
     [echo] Done

phpunit:
  [phpunit] PHPUnit 5.5.4 by Sebastian Bergmann and contributors.
  [phpunit] 
  [phpunit] Runtime:       PHP 5.6.25 with Xdebug 2.4.1
  [phpunit] Configuration: /web/www/com.github/code-smell/php-automate-build-ant/build/phpunit.xml
  [phpunit] 
  [phpunit] ..                                                                  2 / 2 (100%)
  [phpunit] 
  [phpunit] Time: 154 ms, Memory: 6.50MB
  [phpunit] 
  [phpunit] OK (2 tests, 2 assertions)
  [phpunit] 
  [phpunit] Generating code coverage report in Clover XML format ... done
  [phpunit] 
  [phpunit] Generating Crap4J report XML file ... done
  [phpunit] 
  [phpunit] Generating code coverage report in HTML format ... done

phpdox:
   [phpdox] phpDox 0.8.2-dev - Copyright (C) 2010 - 2016 by Arne Blankerts
   [phpdox] 
   [phpdox] [16.10.2016 - 07:46:26] Using config file './phpdox.xml'
   [phpdox] [16.10.2016 - 07:46:26] Registered collector backend 'parser'
   [phpdox] [16.10.2016 - 07:46:26] Registered enricher 'build'
   [phpdox] [16.10.2016 - 07:46:26] Registered enricher 'git'
   [phpdox] [16.10.2016 - 07:46:26] Registered enricher 'checkstyle'
   [phpdox] [16.10.2016 - 07:46:26] Registered enricher 'phpcs'
   [phpdox] [16.10.2016 - 07:46:26] Registered enricher 'pmd'
   [phpdox] [16.10.2016 - 07:46:26] Registered enricher 'phpunit'
   [phpdox] [16.10.2016 - 07:46:26] Registered enricher 'phploc'
   [phpdox] [16.10.2016 - 07:46:26] Registered output engine 'xml'
   [phpdox] [16.10.2016 - 07:46:26] Registered output engine 'html'
   [phpdox] [16.10.2016 - 07:46:26] Starting to process project 'php-automate-build-ant'
   [phpdox] [16.10.2016 - 07:46:26] Starting collector
   [phpdox] [16.10.2016 - 07:46:26] Scanning directory '../src' for files to process
   [phpdox] 
   [phpdox] .                                                 	[1]
   [phpdox] 
   [phpdox] [16.10.2016 - 07:46:26] Saving results to directory '../build/phpdox'
   [phpdox] [16.10.2016 - 07:46:26] Resolving inheritance
   [phpdox] 
   [phpdox] .                                                 	[1]
   [phpdox] 
   [phpdox] [16.10.2016 - 07:46:26] Collector process completed
   [phpdox] 
   [phpdox] [16.10.2016 - 07:46:26] Starting generator
   [phpdox] [16.10.2016 - 07:46:26] Loading enrichers
   [phpdox] [16.10.2016 - 07:46:26] Enricher Build Information initialized successfully
   [phpdox] [16.10.2016 - 07:46:26] Starting event loop.
   [phpdox] 
   [phpdox] .....................                             	[21]
   [phpdox] 
   [phpdox] [16.10.2016 - 07:46:26] Generator process completed
   [phpdox] [16.10.2016 - 07:46:26] Processing project 'php-automate-build-ant' completed.
   [phpdox] 
   [phpdox] 
   [phpdox] Time: 272 ms, Memory: 7.75MB
   [phpdox] 

-check-failure:
     [echo] Checked failure

full-build:
     [echo] Built

BUILD SUCCESSFUL
Total time: 6 seconds    
~~~

#### Build Artifacts

Executing the build.xml script will produce the following build artifacts:

~~~
build
|-- api ...
|-- coverage ...
|-- logs
|   |-- checkstyle.xml
|   |-- clover.xml
|   |-- crap4j.xml
|   |-- jdepend.xml
|   |-- junit.xml
|   |-- phploc.csv
|   |-- phploc.xml
|   |-- pmd-cpd.xml
|   `-- pmd.xml
|-- pdepend ...
`-- phpdox ...
~~~

These artifacts can later be processed by a Continuous Integration server like [Jenkins CI][jenkins]{:target="_blank"}.

### Quick Build

The `quick-build` target installs dependencies, performs a lint check and runs tests without generating code coverage reports. 

~~~bash
$ ant quick-build
~~~

### Static Analysis

The `static-analysis` target installs dependencies and performs static analysis.

~~~bash
$ ant static-analysis
~~~

## Example

You can find a working example - a kind of a template - on [GitHub][repository]{:target="_blank"}.



[ant]: http://ant.apache.org
[composer]: https://getcomposer.org
[git]: https://git-scm.com
[jenkins]: https://jenkins.io
[PhpCodeSniffer]: https://github.com/squizlabs/PHP_CodeSniffer
[PhpCpd]: https://github.com/sebastianbergmann/phpcpd
[PhpDepend]: https://pdepend.org
[PhpDox]: http://phpdox.de
[PhpLoc]: https://github.com/sebastianbergmann/phploc
[PhpMd]: https://phpmd.org
[PhpUnit]: https://phpunit.de
[repository]: https://github.com/code-smell/php-automate-build-ant
