---
layout: post
title: "GemFire Installation on Mac"
categories: [architect, gemfire]
---

When we're using GemFire on laptop, we need to install GemFire. I'm a Mac user, and will install using brew.

## Installation

(Optoinal) If you previously installed GemFire using Homebrew, execute this untap command to ensure a clean update:
```shell
$ brew untap pivotal/tap
```

Retap Pivotal.
```shell
$ brew tap pivotal/tap
```

Install GemFire. In this case, it'll be installed GemFire with the latest version.
```shell
$ brew install gemfire
```

If you are upgrading an existing GemFire installation, execute the brew upgrade command to upgrade GemFire to the latest version:
```shell
$ brew upgrade gemfire
```

Configure your JAVA_HOME environment variable to point to a supported version of Java in your environment variable file like .zshrc, .bash_profile.
```text
JAVA_HOME='/usr/libexec/java_home -v  1.8'
```

## In case of specific version
You can execute below command to install.
```shell
$ brew install gemfire@9.9
```

To delete it, you can execute as below.
```shell
$ brew remove gemfire@9.9
```

Please refere [here](https://gemfire.docs.pivotal.io/910/gemfire/getting_started/installation/install_brew.html){:target="_blank"} for more information.
