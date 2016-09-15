---
layout: default
title: Configuration
---
# Configuration

The build script assumes that the following sets are located in the `build` folder. Of course we could put everything into one file, but by dividing the configuration we make the files more readable by human.

## PHPUnit

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

You can find more information in the [documentation for PHPUnit](https://phpunit.de/manual/current/en/appendixes.configuration.html).

## phpDox

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

You can find more information in the [documentation for phpDox](http://phpdox.de/getting-started.html).

## PHP_CodeSniffer

The `phpcs` and `phpcs-ci` tasks in the `build.xml` assume that an XML configuration file for PHP_CodeSniffer is used to configure the coding standard:

~~~xml
<ruleset name="name-of-your-coding-standard">
    <description>Description of your coding standard</description>
    <rule ref="Generic.PHP.DisallowShortOpenTag"/>
    <!-- ... -->
</ruleset>
~~~

The build script assumes that the rule sets for PHP_CodeSniffer is located at `build/phpcs.xml`.

You can find more information in the [documentation for PHP_CodeSniffer](https://github.com/squizlabs/PHP_CodeSniffer/wiki).

## PHPMD

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

You can find more information in the [documentation for PHPMD](https://phpmd.org/documentation/index.html).

