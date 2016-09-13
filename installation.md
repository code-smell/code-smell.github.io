---
layout: default
title: Installation
---

# Installation

## Globale Tools installieren

Die gesamte Infrastruktur wird auf einem MacBook Pro mit Hilfe von `homebrew` installiert.

### Ant

[Apache Ant][ant]

### Git

[Git Versionsverwaltung][git]

[GitHub][github]


## Jenkins installieren

[Jenkins][jenkins]

## Jenkins Plugins installieren

- [Checkstyle][jkCheckstyle] für das Processing der Logfiles, die von PHP_CodeSniffer im Checkstyle-Format generiert wurden.
- [Clover PHP][jkCloverPHP] für das Processing der Clover XML-Logfiles, die von PHPUNit generiert wurden.
- [Crap4J][jkCrap4J] für das Processing der Crap4J XML Logfiles, die von PHPUnit generiert wurden.
- [DRY][jkDRY] für das Prozessing der Logfiles im PMD-CPD Format, die von PHPCPD generiert wurden.
- [JDepend][jkJDepend] für das Prozessing der Logfiles im JDepend Format, die von PHP_Depend generiert wurden.
- [Plot][jkPlot] für das Prozessing des CSV-Outputs von PHPLOC
- [PMD][jkPMD] für das Prozessing der Logfiles im PMD-Format, die von PHPMD generiert wurden.
- [Violations][jkViolations] für das Prozessing verschiedener Logfiles
- [Warnings][jkWarnings] für das Processing von Warnungen im Konsole Log, die vom PHP-Compiler generiert wurden
- [xUnit][jkXUnit] für das Prozessing von JUnit XML Logfiles
- [Phing][jkPhing] als mögliche Alternative zu Ant

## PHP-Tools installieren

Alle Tools werden global - also nicht projektspezifisch - installiert.

- [Composer][composer]
- [PHP_CodeSniffer][PhpCodeSniffer] tokenizes PHP, JavaScript and CSS files and detects violations of a defined set of coding standards.
- [PHPUnit][PhpUnit] is a programmer-oriented testing framework for PHP. It is an instance of the xUnit architecture for unit testing frameworks.
- [PHP Copy/Paste Detector][PhpCD] is a Copy/Paste Detector (CPD) for PHP code.
- [phpDox][PhpDox] PHP Documentation Generator is the documentation generator for PHP projects.
This includes, but is not limited to, API documentation.
- [PhpDepend][PhpDepend] ?
- [PHPLoc][PhpLoc] A tool for quickly measuring the size of a PHP project.
- [PHPMD][PhpMd] PHP Mess Detector is a source code analyzer.
- [Phing][phing] Phing is a PHP project build system or build tool based on ​Apache Ant.
- []

## Jenkins konfigurieren

### Jenkins > Konfiguration

#### Globale Eigenschaften

	PATH: /Users/thomas/.composer/vendor/bin:/usr/local/bin:/Applications/MAMP/bin/php/php5.6.10/bin:$PATH

#### E-Mail Benachrichtigung

Einstellungen für SMTP-Server vornehmen

### Global Tool Konfiguration

Git Installation:  

- Name geben
- Pfad zu Git definieren

Ant Installation:

- Name geben
- Version festlegen


[git]: https://git-scm.com
[github]: https://github.com
[ant]: http://ant.apache.org
[jenkins]: https://jenkins.io
[jkCheckstyle]: https://wiki.jenkins-ci.org/display/JENKINS/Checkstyle+Plugin
[jkCloverPHP]: https://wiki.jenkins-ci.org/display/JENKINS/Clover+PHP+Plugin
[jkCrap4J]: https://wiki.jenkins-ci.org/display/JENKINS/Crap4J+Plugin
[jkDRY]: http://wiki.jenkins-ci.org/display/JENKINS/DRY+Plugin
[jkHTMLPublisher]: https://wiki.jenkins-ci.org/display/JENKINS/HTML+Publisher+Plugin
[jkJDepend]: https://wiki.jenkins-ci.org/display/JENKINS/JDepend+Plugin  
[jkPlot]: https://wiki.jenkins-ci.org/display/JENKINS/Plot+Plugin
[jkPMD]: https://wiki.jenkins-ci.org/display/JENKINS/PMD+Plugin
[jkViolations]: https://wiki.jenkins-ci.org/display/JENKINS/Violations 
[jkWarnings]: https://wiki.jenkins-ci.org/display/JENKINS/Warnings+Plugin
[jkXUnit]: https://wiki.jenkins-ci.org/display/JENKINS/xUnit+Plugin
[jkPhing]: https://wiki.jenkins-ci.org/display/JENKINS/Phing+Plugin

[PhpCodeSniffer]: https://github.com/squizlabs/PHP_CodeSniffer
[PhpUnit]: https://phpunit.de
[PhpCD]: https://github.com/sebastianbergmann/phpcpd
[PhpDox]: http://phpdox.de
[PhpDepend]: https://pdepend.org
[PhpLoc]: https://github.com/sebastianbergmann/phploc
[PhpMd]: https://phpmd.org
[phing]: https://www.phing.info


