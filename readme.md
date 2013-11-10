# sbt-javafx

sbt-javafx is a plugin for [SBT](http://www.scala-sbt.org/) (Simple Build Tool) for packaging JavaFX applications. It can be used to run and package both Scala- and Java-based applications.

## Quick start

sbt-javafx is available from Maven Central with a groupId of `no.vedaadata` and an artifactId of `sbt-javafx`.

To use it in an SBT project, add an `.sbt` file (e.g. `plugins.sbt`) to the project's `project` directory, with the following content:  

```scala
addSbtPlugin("no.vedaadata" %% "sbt-javafx" % "0.5")
```

A minimal `.sbt` build file (e.g. `build.sbt`) could then look like this:

```scala
name := "my-javafx-application"

version := "1.0"

jfxSettings

JFX.mainClass := Some("mypackage.MyJavaFXApplication")

JFX.devKit := JFX.jdk("C:/Program Files/Java/jdk1.7.0_07")

JFX.addJfxrtToClasspath := true
```

To package the application, simply run the `package-javafx` task.

This is mostly what you need to know. For more details, read on below.

## Examples

Instead of, or in addition to, reading the rest of this document, you might want to take a look at some [examples of using the plugin](https://github.com/kavedaa/sbt-javafx-examples).

Note that these examples lack the `JFX.devKit` setting shown above, since this is of course specific to the system they might be built on.

(It is possible that the examples may not be 100% in sync with the current version of the plugin at any given time, but they should still be illustrative of the concepts.) 

## Enabling the plugin

There are two steps to enabling the plugin:

Adding the following line to an `.sbt` file in the `project` directory:

```scala
addSbtPlugin("no.vedaadata" %% "sbt-javafx" % "0.5-SNAPSHOT")
```

(This makes the plugin available to your project.)

And adding the following line to your build `.sbt` file:

```scala
jfxSettings
```

(This overrides some of SBT's predefined settings in order to enable the plugin's functionality.)  

## Configuring paths to necessary JavaFX files

Two files from the JavaFX SDK are needed by the plugin:

* jfxrt.jar (for compiling and running)
* ant-javafx.jar (for packaging)

The location of the these can be configured in several different ways:

### Using JDK 7u6 or higher

The JDK from version 7u6 and higher has the JavaFX SDK included with it. Specify the path to the JDK root directory like this, e.g.:

```scala
JFX.devKit := JFX.jdk("C:/Program Files/Java/jdk1.7.0_06")
```

For some reason though, even if it is included, jfxrt.jar is *not* added to the Java classpath. So you'll have to add it manually in SBT, however there is a little convenience setting to do that for you:

```scala
JFX.addJfxrtToClasspath := true
```

(With the longish name it might not seem so convenient, but this helps identify it as a temporary *workaround* that will hopefully no longer be needed when JavaFX becomes an official part of Java SE.) 

It is not necessary to add ant-javafx.jar to the classpath.

### Using JDK 8 or higher

From JDK 8 and upwards, you don't need to add `jfxrt.jar` to the classpath (but you still need to specify `JFX.devKit` for the plugin to be able to find the packaging tool). 

### Using a standalone JavaFX SDK

If you're using JDK 6 (which does not include JavaFX SDK), or for some other reason prefer to use the stand-alone JavaFX SDK, specify the path to the root directory like this, e.g.:
 
```scala
JFX.devKit := JFX.sdk("C:/Program Files/Oracle/JavaFX 2.2 SDK")
```

### Manual configuration

You can also specify the full paths to jfxrt.jar and ant-javafx.jar individually, e.g.:

```scala
JFX.jfxrt := Some("C:/Program Files/Java/jdk1.7.0_07/jre/lib/jfxrt.jar")
```
```scala
JFX.antLib := Some("C:/Program Files/Java/jdk1.7.0_07/lib/ant-javafx.jar")
```

(These will take precedence over corresponding paths possibly calculated from specified JDK or SDK directory.)

*Tip:* SBT does not limit you to use only a single .sbt build settings file for your project. Instead, it combines the settings from all .sbt files in your project's root directory. It is a good idea to keep the path configuration settings in a seperate file than the main build file and exclude this file from version control, especially when you're collaborating with others on the project or for other reasons compiling it on several different machines where the paths may not be the same.  (Note that SBT seems to load the `.sbt` files in alphabetical order, which sometimes matters...)

## Running from within SBT

SBT is not able to launch a `javafx.application.Application` on its own. It needs a class with a static `main` method for its `run` task, which name must be configured in the `mainClass` setting (which is distinct from `JFX.mainClass`).

By default, the plugin will set SBT's `mainClass` to the same value as `JFX.mainClass`. This makes it simple to add the necessary launcher code.

### Scala-based applications

You can use a companion object with a `main` method that has code to launch the JavaFX application, e.g.:

```scala
object MyJavaFXApplication {
  def main(args: Array[String]) {
    Application launch(classOf[MyJavaFXApplication], args: _*)
  }
}
```
 
If you for some reason would want to name this differently, you can override SBT's `mainClass` like this, e.g.:

```scala
mainClass in (Compile, run) := Some("some.other.Launcher")
```

### Java-based applications

You can add a static `main` method to your JavaFX application class, e.g.:


```java
public class MyJavaFXApplication extends Application {

	public static void main(String[] args) {
		Application.launch(args);
	}
	
	// rest of application here
}
```

## Packaging

A JavaFX application must have a main class that extends from `javafx.application.Application`, e.g.:

```scala
class MyJavaFXApplication extends Application {

	//	application here
}
```

The name of this class must be configured like this:

```scala
JFX.mainClass := Some("mypackage.MyJavaFXApplication")
```

Execute the `package-javafx` task to package the application.

The packaged application will reside inside `target/<scala-version>/<artifact-name>/`, e.g. `target/scala_2.9.2/my-javafx-application_2.9.2-1.0/`. (It is possible to customize the name of the directory.)

The application will be identical to one packaged with JavaFX's Ant tools, e.g. by using the Netbeans IDE. It will contains, at least, a `.jar` fil, a `.jnlp` file, an `.html` file, as well as a `lib/` directory with all library jars added via any of SBT's library management methods, including the standard Scala library.

### Native bundles

For creating so-called "native bundles", that is, self-contained applications which co-bundles the JRE, use this setting:

```scala
JFX.nativeBundles := bundleType	// where bundleType is a String
```
A typical value for the bundle type is `"image"`.  See the [JavaFX documentation](http://docs.oracle.com/javafx/2/deployment/self-contained-packaging.htm) for possible values and further information.

#### Using the correct Java version

Self-contained applications must be packaged using the JDK version of the JRE and not the stand-alone JRE. (If you have installed the JDK you will probably have both.) If you use the JRE version, you will get an error message saying "jvm.dll is not found" (on Windows, probably similar on other platforms).

This means that SBT must be started with the JDK version of the JRE. This can be assured by setting JAVA_HOME to the correct path, either globally or within SBT's `sbt.bat` startup file.

### Java-only applications

It is very much possible to use the plugin to package applications written in Java. If your application uses no Scala code at all, you might want to use the `javaOnly` setting:

```scala
JFX.javaOnly := true
```

This is a convenience setting that excludes the standard Scala library from being packaged with the application, and makes the output path a bit simpler, so that it becomes e.g. `target/my-javafx-application-1.0/`.

## Other settings

### Signing

```scala
JFX.elevated := true

JFX.keyStore := Some("path/to/keystore")

JFX.storePass := Some("mypassword")

JFX.alias := Some("myalias")

JFX.keyPass := Some("mykeypass")
```

### Application info

```scala
JFX.vendor := "ACME Inc."

JFX.title := "Rocket Launcher"   

JFX.category := "Mission critical"

JFX.description := "Launches rockets"

JFX.copyright := "ACME 2013"

JFX.license := "ACME"
```
