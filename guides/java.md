# 3 billion devices run Java, do you?

__Java__ usually refers to the programming language but also to the platform: a
set of tools -virtual machine, compiler and libraries- which allow developers
to create cross-platform applications under the concept of *write once, run
anywhere*.

Despite the initial complexity of the Java ecosystem, it's important to
understant that there is only one set of source code for the JDK released under
GPL license and hosted at [OpenJDK](http://openjdk.java.net/projects/jdk/). You
can follow [these
instructions](http://hg.openjdk.java.net/jdk9/jdk9/raw-file/tip/common/doc/building.html)
to compile and generate your own JDK flavour.

Although it sounds scary, is surprisingly easy and takes less than 1 hour! But
that's not the pourpose of this tutorial, so downloading a build seems like a
good idea.

# OpenJDK, Oracle JDK, AdoptOpenJDK.. help!

Different vendors build the OpenJDK adding additional tools, utilities or
branding elements, but never modifing the language. As result the vendor
provides a new build with some unique vendor capabilities or certification
processes. 

There are many JDK implementations, but the most used ones are:

* [OpenJDK](http://jdk.java.net/). GPL license unbranded builds of the OpenJDK by Oracle.
* [Oracle JDK](http://www.oracle.com/technetwork/java/javase/downloads/). Branded builds from Oracle that could be used without cost.

[Zulu](https://www.azul.com/downloads/zulu/), [IBM
JDK](https://developer.ibm.com/javasdk/support/lifecycle/), [Red
Hat](https://developers.redhat.com/products/openjdk/overview/) or
[AdoptOpenJDK](https://adoptopenjdk.net/) are other examples of Java
implementations..

# Installation

All the examples are based on *Oracle JDK version 8*, so let's see how to set up the environment.

## Linux / Windows

Go to [Oracle Java website](https://java.com/en/download/manual.jsp), accept
the license and download the tarball (Version 8 Update 191 at the time of
writting this).

Make sure the new Java installation is the default Java in your system:

* On Linux, use `update-alternatives --config java` whenever is possible.
* Add `bin` directory to your PATH environment.

## macOS

We recommend to use __brew__ if possible. If you haven't installed brew already [install it!](http://brew.sh/) 

Then install __java8__ as follows:

```bash
$ brew cask uninstall java
$ brew tap caskroom/versions
$ brew cask install java8
```

# Setting up the environment

Managing a Java project and its dependencies manually could be an exhausting
task if you don't use the proper tools or IDEs. There are several tools which
makes us life eaiser. The most interesting ones are
[Gradle](https://gradle.org/) and [maven](https://maven.apache.org/).

Although our `Java SDK` supports both `Gradle` and `maven`, for our example we
are going to use Gradle. Let's see how it works.

## Installing Gradle

Go ahead and install Gradle following the
[instructions](https://gradle.org/install/). Make sure the `gradle` tool belongs to
your `PATH` variable.

To test the Gradle installation, run Gradle from the command-line:

```
gradle

Welcome to Gradle 4.10.2!

Here are the highlights of this release:
 - Incremental Java compilation by default
 - Periodic Gradle caches cleanup
 - Gradle Kotlin DSL 1.0-RC6
 - Nested included builds
 - SNAPSHOT plugin versions in the `plugins {}` block

For more details see https://docs.gradle.org/4.10.2/release-notes.html
```

## Hello world!

1. Create a folder on your `$HOME` directory called `JavaTest`.
2. Switch to `JavaTest` and create a subdirectry structure that will host our project: `src/main/java/hello` 

If you are on Linux or macOS, just type:

```bash
mkdir -p src/main/java/hello
```

3. Switch to `src/main/java/hello` and create the following `HelloWorld.java` file:

```java
package hello;

public class HelloWorld {
  public static void main(String[] args) {
    System.out.println("Hello World");
  }
}
```

4. Go to the root folder of your project `JavaTest` and create a file called `build.gradle` with the following content:

```
apply plugin: 'java'
apply plugin: 'application'

mainClassName = 'HelloWorld'
```

5. On the root folder of your project, build the source using Gradle:

```bash
gradle build

BUILD SUCCESSFUL in 2s
2 actionable tasks: 2 executed
```

A new folder `build/libs` has been created containing the `jar` file of your project.

## Gradle Wrapper

The Gradle wrapper is the best way to start building and running your projects. Let's learn how it works:

1. Go to your root folder `JavaTest` and run:

```bash
gradle wrapper
```

You'll notice a few new files in your root folder. You can freely add them to your
version control system so everyone can build it just the same way.

2. Build the source code using the wrapper:

```bash
./gradlew build

BUILD SUCCESSFUL in 4s
2 actionable tasks: 2 up-to-date
```

3. And finally execute it:

```bash
./gradlew run

Hello world
```

# Integrating Amadeus SDK

So far we have built a very simple piece of code. Let's do something cool by
calling one of our [APIs](https://developers.amadeus.com) from your Java code.

According to our [Java SDK](https://github.com/amadeus4dev/amadeus-java) documentation, we need to update our `build.gradle` file to include the following dependencies:

```
compile 'com.google.code.gson:gson:2.8.5'
compile "com.amadeus:amadeus-java:1.1.2"
```

Our new `build.gradle` will look like:

```
apply plugin: 'java'
apply plugin: 'application'

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile 'com.google.code.gson:gson:2.8.5'
    compile 'com.amadeus:amadeus-java:1.1.2'
}

repositories { maven { url "https://jcenter.bintray.com" } }

jar {
    baseName = 'amadeusExample'
    version =  '0.1.0'
}

mainClassName = 'AmadeusExample'
```

Replace the previously created `HelloWorld.java` with the file `AmadeusExample.java`:

```java
import com.amadeus.Amadeus;
import com.amadeus.Params;

import com.amadeus.exceptions.ResponseException;

import com.amadeus.referenceData.Airlines;
import com.amadeus.resources.Airline;

public class AmadeusExample {
  public static void main(String[] args) throws ResponseException {
    Amadeus amadeus = Amadeus
            .builder("YOU_CLIENT_ID","YOUR_CLIENT_SECRET")
            .build();

    Airline[] airlines = amadeus.referenceData.airlines.get(Params
      .with("IATACode", "BA")
      .and("ICAOCode", "AIC"));

    System.out.println(airlines[0]);
  }
}
```
