# Reproducer for optional dependency issue


## Background
It appears that the gluonfx maven plugin is adding optional dependencies of dependencies to the classpath, when it should only be including those if they are explicit separate dependencies. Per the [maven documentation on optional dependencies](https://maven.apache.org/guides/introduction/introduction-to-optional-and-excludes-dependencies.html):

> If a user wants to use functionality related to an optional dependency, they have to redeclare that optional dependency in their own project.

It appears that the functionality is included by gluonfx even in the absence of a redeclaration.

This application was created using the Gluon Mobile Eclipse plugin (single view project). The versions of the various dependencies and plugins have been updated to their latest versions. I added logback-classic 1.2.8 as a dependency, which in turn has groovy as an optional dependency. Notably, it does NOT redeclare the groovy dependency. Yet groovy-2.4.0.jar ends up on the classpath. (Note this example does not actually use logback for anything; it is just included in the pom.)

In my real application, this caused an issue because there was another dependency (picocli) that checked for the presence of groovy and executed a different code path if it was present. This 

I have tried building with these JDKs:
* Gluon GraalVM 22.0.0.3 JDK 17 (windows)
* Gluon GraalVM 22.0.0.3 JDK 11 (linux targeting android)
* Gluon GraalVM 22.0.0.3 JDK 17 (linux targeting android)

(Note that as of logback-classic 1.2.9, groovy is no longer an optional dependency and thus this problem goes away. However, this was a convenient way to demonstrate the issue.)

## Reproducing the issue
This is a maven project, so on windows I do this from a VS Native 64-bit Prompt:

```
mvn gluonfx:build
```

One of the first things that is printed is the classpath; near the bottom you can see groovy-2.4.0.jar is included.

```
[Mon Feb 14 16:50:53 EST 2022][INFO] [SUB] 'C:\Users\bob.marinier\git\gluonfxoptionaldependencytest\target\classes;C:\Users\bob.marinier\git\gluonfxoptionaldependencytest\target\gluonfx\x86_64-windows\gvm\tmp\deps\logback-classic-1.2.8.jar;C:\Users\bob.marinier\git\gluonfxoptionaldependencytest\target\gluonfx\x86_64-windows\gvm\tmp\deps\logback-core-1.2.8.jar;C:\Users\bob.marinier\git\gluonfxoptionaldependencytest\target\gluonfx\x86_64-windows\gvm\tmp\deps\charm-glisten-6.0.6.jar;C:\Users\bob.marinier\git\gluonfxoptionaldependencytest\target\gluonfx\x86_64-windows\gvm\tmp\deps\display-4.0.13-desktop.jar;C:\Users\bob.marinier\git\gluonfxoptionaldependencytest\target\gluonfx\x86_64-windows\gvm\tmp\deps\display-4.0.13.jar;C:\Users\bob.marinier\git\gluonfxoptionaldependencytest\target\gluonfx\x86_64-windows\gvm\tmp\deps\lifecycle-4.0.13-desktop.jar;C:\Users\bob.marinier\git\gluonfxoptionaldependencytest\target\gluonfx\x86_64-windows\gvm\tmp\deps\lifecycle-4.0.13.jar;C:\Users\bob.marinier\git\gluonfxoptionaldependencytest\target\gluonfx\x86_64-windows\gvm\tmp\deps\statusbar-4.0.13.jar;C:\Users\bob.marinier\git\gluonfxoptionaldependencytest\target\gluonfx\x86_64-windows\gvm\tmp\deps\storage-4.0.13-desktop.jar;C:\Users\bob.marinier\git\gluonfxoptionaldependencytest\target\gluonfx\x86_64-windows\gvm\tmp\deps\storage-4.0.13.jar;C:\Users\bob.marinier\git\gluonfxoptionaldependencytest\target\gluonfx\x86_64-windows\gvm\tmp\deps\util-4.0.13.jar;C:\Users\bob.marinier\git\gluonfxoptionaldependencytest\target\gluonfx\x86_64-windows\gvm\tmp\deps\javafx.base.jar;C:\Users\bob.marinier\git\gluonfxoptionaldependencytest\target\gluonfx\x86_64-windows\gvm\tmp\deps\javafx.controls.jar;C:\Users\bob.marinier\git\gluonfxoptionaldependencytest\target\gluonfx\x86_64-windows\gvm\tmp\deps\javafx.graphics.jar;C:\Users\bob.marinier\git\gluonfxoptionaldependencytest\target\gluonfx\x86_64-windows\gvm\tmp\deps\slf4j-api-1.7.32.jar;C:\Users\bob.marinier\git\gluonfxoptionaldependencytest\target\gluonfx\x86_64-windows\gvm\tmp\deps\activation-1.1.jar;C:\Users\bob.marinier\git\gluonfxoptionaldependencytest\target\gluonfx\x86_64-windows\gvm\tmp\deps\janino-3.0.6.jar;C:\Users\bob.marinier\git\gluonfxoptionaldependencytest\target\gluonfx\x86_64-windows\gvm\tmp\deps\groovy-2.4.0.jar;C:\Users\bob.marinier\git\gluonfxoptionaldependencytest\target\gluonfx\x86_64-windows\gvm\tmp\deps\mail-1.4.jar;C:\Users\bob.marinier\git\gluonfxoptionaldependencytest\target\gluonfx\x86_64-windows\gvm\tmp\deps\activation.jar;C:\Users\bob.marinier\git\gluonfxoptionaldependencytest\target\gluonfx\x86_64-windows\gvm\tmp\deps\javax.servlet-api-3.1.0.jar;C:\Users\bob.marinier\git\gluonfxoptionaldependencytest\target\gluonfx\x86_64-windows\gvm\tmp\deps\commons-compiler-3.0.6.jar;C:\Users\bob.marinier\.m2\repository\com\gluonhq\substrate\0.0.52\substrate-0.0.52.jar;C:\Users\bob.marinier\git\gluonfxoptionaldependencytest\target\gluonfx\x86_64-windows\gvm\tmp\classpathJar.jar;C:\Users\bob.marinier\graalvm-svm-java17-windows-gluon-22.0.0.3-Final\lib\svm\library-support.jar' \

```
