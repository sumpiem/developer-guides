# 3 billion devices run Java, do you?

*Java* usually refers to the programming language but also to the platform: a
set of tools -virtual machine, compiler and libraries- which allow developers
to create cross-platform applications under the concept of *write once, run
anywhere*.

Despite the complexity of the Java ecosystem, it's important to understand that
there is only one set of source code for the JDK released under GPL license and
hosted at [OpenJDK](http://openjdk.java.net/projects/jdk/). You can follow
[these
instructions](http://hg.openjdk.java.net/jdk9/jdk9/raw-file/tip/common/doc/building.html)
to compile and generate your own JDK flavour.

Although it sounds scary, is surprisingly easy and takes less than 1 hour! But
that's not the purpose of this tutorial, so downloading a build seems like a
good idea.

# OpenJDK? Oracle JDK? help!

Different vendors build the OpenJDK adding additional tools, utilities or
branding elements, but never modifing the language. As result the vendor
provides a new build with some unique vendor capabilities or certification
processeses. 

There are many JDK implementations, but the most used ones are:

* [OpenJDK](http://jdk.java.net/). Oracle GPL license unbranded builds of the OpenJDK.
* [Oracle JDK](http://www.oracle.com/technetwork/java/javase/downloads/). Branded builds from Oracle that could be used without cost.

But there are more implementations like [Zulu](https://www.azul.com/downloads/zulu/), [IBM
JDK](https://developer.ibm.com/javasdk/support/lifecycle/), [Red
Hat](https://developers.redhat.com/products/openjdk/overview/) or
[AdoptOpenJDK](https://adoptopenjdk.net/).

# Installation

All the examples are based on __Oracle JDK version 8__, so let's see how to install it.

## Linux / Windows

Go to [Oracle Java website](https://java.com/en/download/manual.jsp), accept
the license and download the tarball (Version 8 Update 191 at the time of
writting this).

Make sure the new Java installation is the default Java in your system:

* On Linux, use `update-alternatives --config java` whenever is possible.
* Add `bin` directory to your PATH environment.
* Export a new `JAVA_HOME` environment variable pointing to the directory where Java has been installed.

## macOS

We recommend to use __brew__ if possible. If you haven't installed brew already [install it!](http://brew.sh/) 

Then install __Java 8__ as follows:

```bash
brew cask uninstall java
brew tap caskroom/versions
brew cask install java8
```

# Setting up the environment

Managing a Java project and its dependencies manually could be an exhausting
task if you don't use the proper tools or IDEs (Eclipse, NetBeans,
IntelliJ...). This tutorial is meant to be followed using command line. This
will give us a better understanding about what's going on beneath any IDE. 

There are several tools which
makes us life eaiser. The most interesting ones are
[Gradle](https://gradle.org/) and [Maven](https://maven.apache.org/).

Although our [Java SDK](https://github.com/amadeus4dev/amadeus-java) supports
both `Gradle` and `Maven`, for our example we are going to use `Gradle`. Let's
see how it works.

## Installing Gradle

Go ahead and install Gradle following the
[instructions](https://gradle.org/install/) from the website. Make sure the
`gradle` tool belongs to your `PATH` variable.

To test the Gradle installation, run `gradle` from the command line:

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

## Hello world Java!

> We will use Unix-based commands. Windows has similar commands for each.

1. Create a new folder `JavaTest`  on your `$HOME` directory called.
2. Switch to `JavaTest` and create a subdirectry structure that will host our project: `src/main/java/hello` 

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

4. Go back to the root folder of your project `JavaTest` and initialice your project using `gradle`:

```bash
gradle init
```

You'll notice a few new files in your root folder. You can freely add them to your
version control system so everyone can build it just the same way.

5. Edit the `build.gradle` file with your favourite editor and add:

```
apply plugin: 'java'
apply plugin: 'application'

mainClassName = 'hello.HelloWorld'
```

> Note that mainClassName must be fully qualified class name.

6. Let's build the source using the `gradlew` wrapper script that Gradle has just created:

```bash
./gradlew build

BUILD SUCCESSFUL in 2s
2 actionable tasks: 2 executed
```

The `build` argument is a task. Each project includes a collection of tasks,
each of which performs a basic operation. Gradle comes with a set of predefined
tasks that could be extended using an API. Managing tasks is out of the scope
of this tutorial, but you can find more information about how to extend and
configure tasks on [Gradle documentation](https://guides.gradle.org/). 

A new folder `build/libs` has been created containing the `jar` file of your project.

7. And finally let's run it:

```bash
./gradlew run

> Task :run
Hello World

BUILD SUCCESSFUL in 0s
2 actionable tasks: 1 executed, 1 up-to-date
```

# Integrating Amadeus SDK

So far we have built a very simple piece of code. Let's do something cool by
calling one of our [Flight Search APIs](https://developers.amadeus.com) from
your Java code.

According to our [Java SDK](https://github.com/amadeus4dev/amadeus-java)
documentation, we need to update our `build.gradle` file to include the
following dependencies:

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

Note that we have added `repositories` in order to retrieve the `Amadeus Java
SDK` jar file automtically during the `build` task.

It's time to call the APIs from our Java sample. Replace the previously
created `HelloWorld.java` with the file `AmadeusExample.java`:

```java
import com.amadeus.Amadeus;
import com.amadeus.Params;

import com.amadeus.exceptions.ResponseException;

import com.amadeus.shopping.FlightDestinations;
import com.amadeus.resources.FlightDestination;

public class AmadeusExample {
  public static void main(String[] args) throws ResponseException {
    Amadeus amadeus = Amadeus
            .builder("YOU_CLIENT_ID","YOUR_CLIENT_SECRET")
            .build();

    FlightDestination[] flightDestinations = amadeus.shopping.flightDestinations.get(Params.with("origin", "MAD"));

    if (flightDestinations[0].getResponse().getStatusCode() != 200) {
        System.out.println("Wrong status code for Flight Inspiration Search: " + flightDestinations[0].getResponse().getStatusCode());
        System.exit(-1);
    }
    
    System.out.println(flightDestinations[0]);
  }
}
```

The sample calls [Flight Inspiration Search
API](https://developers.amadeus.com/self-service/category/203/api-doc/3) in
order to retrieve best offers departing Madrid. 

Finally, let's execute the sample:

```bash
./gradlew run

> Task :run
FlightDestination(type=flight-destination, origin=MAD, destination=BIO, departureDate=Sat Nov 17 00:00:00 CET 2018, returnDate=Wed Nov 21 00:00:00 CET 2018, price=FlightDestination.Price(total=92.26))

```

# What's next?

During this tutorial we have learnt how to set up a Java project from scratch and how to make a
first API call using the Amadeus Java SDK. But the possibilities are huge given
the rich Java ecosystem (Android, web or console applications). Let your imagination fly!


