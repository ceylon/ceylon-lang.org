---
title: Installing Ceylon IDE for IntelliJ
layout: intellij13
tab: documentation
unique_id: docspage
author: Bastien Jansen
---
# #{page.title}

The 1.3.3 release of the Ceylon IDE for IntelliJ is available from JetBrains' 
plugins site. Note that the version of the Ceylon compiler embedded 
in this release is compatible with the Ceylon 1.3.3 command line tools.

## IntelliJ platform

You'll need an installation of [IntelliJ 2017.x][intellij], or of 
[Android Studio 2.x][studio].

## Java 7+ requirement

While the IDE itself uses Java 8, Ceylon IDE for IntelliJ supports projects
based on Java 7 or Java 8. Projects are not compatible with Java 6 or earlier.

## Installing Ceylon IDE for IntelliJ

Here's what you need to do to install Ceylon IDE:

1. Start IntelliJ or Android Studio.
2. If you already have any opened project, go to `File > Settings (on Mac, Preferences) > Plugins` in the main menu.<br/>
   If you have just started, use Configure > Plugins menu at the startup window.
3. Click `Browse repositories...` button, then type `Ceylon IDE` in the search box and click Install.<br/>
   <img src="/images/screenshots/intellij/installation.png" alt="Plugin repository" width="80%" height="auto"/>
4. After the plugin is downloaded, restart the IDE when prompted.

[intellij]: https://www.jetbrains.com/idea/
[studio]: https://developer.android.com/studio/index.html
[issues]: https://github.com/eclipse/ceylon-ide-intellij/issues
[repositories]: https://www.jetbrains.com/idea/help/managing-enterprise-plugin-repositories.html

## Installing development builds and nightly builds

If you want to try the latest build of Ceylon IDE before it is released, you
can use alternative plugin repositories. Inside IntelliJ, follow [these instructions][repositories]
to add one of the following custom repository URLs: 

* Development builds (published irregularly) <br/>
  <http://downloads.ceylon-lang.org/ide/intellij/development/updatePlugins.xml>
* Nightly builds <br/>
  <https://ci-ceylon.rhcloud.com/job/ceylon-ide-intellij/ws/out/installation-packages/updatePlugins.xml>

Beware that these are unstable and unfinished versions.

## Issues

Please [report any issue in our issue tracker][issues].
