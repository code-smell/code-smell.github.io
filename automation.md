---
layout: default
title: Automation
---

# Automation

Thanks to [Apache Ant](http://ant.apache.org) for automating our build process. It orchestrates the execution of the tools defined in our `build.xml` build script.

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<project name="name-of-project" default="full-build">

    <!-- We assume that all tools are on the $PATH -->
    <property name="pdepend" value="pdepend"/>
    <property name="phpcpd" value="phpcpd"/>
    <property name="phpcs" value="phpcs"/>
    <property name="phpdox" value="phpdox"/>
    <property name="phploc" value="phploc"/>
    <property name="phpmd" value="phpmd"/>
    <property name="phpunit" value="phpunit"/>

    <target name="full-build"
            depends="prepare,composer,static-analysis,phpunit,phpdox,-check-failure"
            description="Performs static analysis, runs the tests, and generates project documentation">
        <echo message="Built"/>
    </target>

    <target name="full-build-parallel"
            depends="prepare,composer,static-analysis-parallel,phpunit,phpdox,-check-failure"
            description="Performs static analysis (executing the tools in parallel), runs the tests, and generates project documentation"/>

    <target name="quick-build"
            depends="prepare,composer,lint,phpunit-no-coverage"
            description="Performs a lint check and runs the tests (without generating code coverage reports)"/>

    <target name="static-analysis"
            depends="lint,phploc-ci,pdepend,phpmd-ci,phpcs-ci,phpcpd-ci"
            description="Performs static analysis">
        <echo message="Done"/>
    </target>

    <!-- Adjust the threadCount attribute's value to the number of CPUs -->
    <target name="static-analysis-parallel"
            description="Performs static analysis (executing the tools in parallel)">
        <parallel threadCount="2">
            <sequential>
                <antcall target="pdepend"/>
                <antcall target="phpmd-ci"/>
            </sequential>
            <antcall target="lint"/>
            <antcall target="phpcpd-ci"/>
            <antcall target="phpcs-ci"/>
            <antcall target="phploc-ci"/>
        </parallel>
    </target>

    <target name="clean"
            unless="clean.done"
            description="Cleanup build artifacts">
        <delete dir="${basedir}/build/api"/>
        <delete dir="${basedir}/build/coverage"/>
        <delete dir="${basedir}/build/logs"/>
        <delete dir="${basedir}/build/pdepend"/>
        <delete dir="${basedir}/build/phpdox"/>
        <property name="clean.done" value="true"/>
    </target>

    <target name="prepare"
            unless="prepare.done"
            depends="clean"
            description="Prepare for build">
        <mkdir dir="${basedir}/build/api"/>
        <mkdir dir="${basedir}/build/coverage"/>
        <mkdir dir="${basedir}/build/logs"/>
        <mkdir dir="${basedir}/build/pdepend"/>
        <mkdir dir="${basedir}/build/phpdox"/>
        <property name="prepare.done" value="true"/>
    </target>

    <target name="lint"
            unless="lint.done"
            description="Perform syntax check of sourcecode files">
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

    <target name="phploc"
            unless="phploc.done"
            description="Measure project size using PHPLOC and print human readable output. Intended for usage on the command line.">
        <exec executable="${phploc}" taskname="phploc">
            <arg value="--count-tests"/>
            <arg path="${basedir}/src"/>
            <arg path="${basedir}/tests"/>
        </exec>

        <property name="phploc.done" value="true"/>
    </target>

    <target name="phploc-ci"
            unless="phploc.done"
            depends="prepare"
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

    <target name="pdepend"
            unless="pdepend.done"
            depends="prepare"
            description="Calculate software metrics using PHP_Depend and log result in XML format. Intended for usage within a continuous integration environment.">
        <exec executable="${pdepend}" taskname="pdepend">
            <arg value="--jdepend-xml=${basedir}/build/logs/jdepend.xml"/>
            <arg value="--jdepend-chart=${basedir}/build/pdepend/dependencies.svg"/>
            <arg value="--overview-pyramid=${basedir}/build/pdepend/overview-pyramid.svg"/>
            <arg path="${basedir}/src"/>
        </exec>

        <property name="pdepend.done" value="true"/>
    </target>

    <target name="phpmd"
            unless="phpmd.done"
            description="Perform project mess detection using PHPMD and print human readable output. Intended for usage on the command line before committing.">
        <exec executable="${phpmd}" taskname="phpmd">
            <arg path="${basedir}/src"/>
            <arg value="text"/>

            <arg path="${basedir}/build/phpmd.xml"/>
        </exec>

        <property name="phpmd.done" value="true"/>
    </target>

    <target name="phpmd-ci"
            unless="phpmd.done"
            depends="prepare"
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

    <target name="phpcs"
            unless="phpcs.done"
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

    <target name="phpcs-ci"
            unless="phpcs.done"
            depends="prepare"
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

    <target name="phpcpd"
            unless="phpcpd.done"
            description="Find duplicate code using PHPCPD and print human readable output. Intended for usage on the command line before committing.">
        <exec executable="${phpcpd}" taskname="phpcpd">
            <arg path="${basedir}/src"/>
        </exec>

        <property name="phpcpd.done" value="true"/>
    </target>

    <target name="phpcpd-ci"
            unless="phpcpd.done"
            depends="prepare"
            description="Find duplicate code using PHPCPD and log result in XML format. Intended for usage within a continuous integration environment.">
        <exec executable="${phpcpd}" taskname="phpcpd">
            <arg value="--log-pmd"/>
            <arg path="${basedir}/build/logs/pmd-cpd.xml"/>
            <arg path="${basedir}/src"/>
        </exec>

        <property name="phpcpd.done" value="true"/>
    </target>

    <target name="phpunit"
            unless="phpunit.done"
            depends="prepare"
            description="Run unit tests with PHPUnit">
        <exec executable="${phpunit}" resultproperty="result.phpunit" taskname="phpunit">
            <arg value="--configuration"/>
            <arg path="${basedir}/build/phpunit.xml"/>
            <arg path="tests"/>
        </exec>

        <property name="phpunit.done" value="true"/>
    </target>

    <target name="phpunit-no-coverage"
            unless="phpunit.done"
            depends="prepare"
            description="Run unit tests with PHPUnit (without generating code coverage reports)">
        <exec executable="${phpunit}" failonerror="true" taskname="phpunit">
            <arg value="--configuration"/>
            <arg path="${basedir}/build/phpunit.xml"/>
            <arg value="--no-coverage"/>
        </exec>

        <property name="phpunit.done" value="true"/>
    </target>

    <target name="phpdox"
            unless="phpdox.done"
            depends="phploc-ci,phpcs-ci,phpmd-ci"
            description="Generate project documentation using phpDox">
        <exec executable="${phpdox}" dir="${basedir}/build" taskname="phpdox"/>

        <property name="phpdox.done" value="true"/>
    </target>

    <target name="composer" description="Installing composer dependencies">
        <exec executable="composer" failonerror="true">
            <arg value="install"/>
            <arg value="--prefer-dist"/>
            <arg value="--no-progress"/>
        </exec>
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

The build script assumes that the rule sets for PHP_CodeSniffer and PHPMD are located at `build/phpcs.xml` and `build/phpmd.xml`.

~~~
build
|-- phpcs.xml
|-- phpdox.xml
|-- phpmd.xml
`-- phpunit.xml
build.xml
~~~

Use the following properties if the tools are located as PHARs in `${basedir}/build/tools`.

~~~xml
<property name="pdepend" value="${basedir}/build/tools/pdepend.phar"/>
<property name="phpcpd"  value="${basedir}/build/tools/phpcpd.phar"/>
<property name="phpcs"   value="${basedir}/build/tools/phpcs.phar"/>
<property name="phpdox"  value="${basedir}/build/tools/phpdox.phar"/>
<property name="phploc"  value="${basedir}/build/tools/phploc.phar"/>
<property name="phpmd"   value="${basedir}/build/tools/phpmd.phar"/>
<property name="phpunit" value="${basedir}/build/tools/phpunit.phar"/>
~~~

Use the following properties if the tools are managed by Composer in `${basedir}/vendor/bin`.

~~~xml
<property name="pdepend" value="${basedir}/vendor/bin/pdepend"/>
<property name="phpcpd"  value="${basedir}/vendor/bin/phpcpd"/>
<property name="phpcs"   value="${basedir}/vendor/bin/phpcs"/>
<property name="phpdox"  value="${basedir}/vendor/bin/phpdox"/>
<property name="phploc"  value="${basedir}/vendor/bin/phploc"/>
<property name="phpmd"   value="${basedir}/vendor/bin/phpmd"/>
<property name="phpunit" value="${basedir}/vendor/bin/phpunit"/>
~~~

Here is an overview of the tasks defined in the above build.xml (download) script that are intended to be directly invoked:

- full-build is the default build target. It invokes all the tools in such a way that XML logfiles are written to default locations. Running the full build may take a considerable amount of time and might only be useful to perform nightly.
- full-build-parallel is the same as full-build (see above) except that it executes the static analysis tools in parallel. Even when leveraging multiple CPUs, running the full build may take a considerable amount of time and might only be useful to perform nightly.
- quick-build is a build target intended to be run by jobs that are triggered for every push to a repository. It performs a lint check and runs the tests (without generating code coverage reports).
- clean can be used to clean up (delete) all build artifacts (logfiles, etc. that are produced during the build).
- lint can be used to perform a syntax check of the project sources using php -l. This task can be used before committing.
- phpcs can be used to find coding standard violations and print human readable output. This task can be used before committing.
- phpcpd can be used to find duplicate code and print human readable output. This task can be used before committing.
- phpmd can be used to perform project mess detection and print human readable output. This task can be used before committing.
- phpunit can be used to run unit tests. This task can be used before committing.
- phpdox can be used to generate project documentation.
- phploc can be used to measure the project size.

The other tasks can, of course, also be invoked directly but that is not their intended purpose. They are invoked by the tasks listed above.

Executing the build.xml script will produce the following build directory:

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

These build artifacts will be processed by Jenkins.
