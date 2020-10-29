---
layout: post
title: GemFire Installation on Mac
#subtitle: Excerpt from Soulshaping by Jeff Brown
gh-repo: hshin-pivotal/hshin-pivotal.github.io
gh-badge: [star, fork, follow]
#cover-img: /assets/img/path.jpg
#thumbnail-img: /assets/img/thumb.png
#share-img: /assets/img/path.jpg
tags: [Architect, GemFire]
comments: true
---

When we're using GemFire on laptop, we need to install GemFire. I'm a Mac user, and will install using brew.

## Installation

(Optoinal) If you previously installed GemFire using Homebrew, execute this untap command to ensure a clean update:
~~~
$ brew untap pivotal/tap
~~~

Retap Pivotal.
~~~
$ brew tap pivotal/tap
~~~

Install GemFire. In this case, it'll be installed GemFire with the latest version.
```shell
$ brew install gemfire
```

If you are upgrading an existing GemFire installation, execute the brew upgrade command to upgrade GemFire to the latest version:
~~~
$ brew upgrade gemfire
~~~

Configure your JAVA_HOME environment variable to point to a supported version of Java in your environment variable file like .zshrc, .bash_profile.
~~~
JAVA_HOME='/usr/libexec/java_home -v  1.8'
~~~

## In case of specific version
You can execute below command to install.
~~~
$ brew install gemfire@9.9
~~~

To delete it, you can execute as below.
~~~
$ brew remove gemfire@9.9
~~~

Please refere [here](https://gemfire.docs.pivotal.io/910/gemfire/getting_started/installation/install_brew.html) for more information.
