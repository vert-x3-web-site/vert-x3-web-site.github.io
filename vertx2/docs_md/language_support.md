<!--
This work is licensed under the Creative Commons Attribution-ShareAlike 3.0 Unported License.
To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/3.0/ or send
a letter to Creative Commons, 444 Castro Street, Suite 900, Mountain View, California, 94041, USA.
-->

[TOC]

The core Vert.x distribution doesn't know about any languages other than Java. All language support in Vert.x is provided in the form of modules.

Before embarking on implementing a language implementation module, please get familiar with [Vert.x modules](mods_manual.html).

It's also highly recommended you write your module using the [Vert.x Gradle Template Project](gradle_dev.html) or the [Vert.x Maven Archetype](maven_dev.html).

This document provides an outline for how you can provide support for a new language (or an alternative to an existing language with a different language engine) in Vert.x

# Implement `VerticleFactory`

You should implement the interface `org.vertx.java.platform.VerticleFactory` in your module. The Vert.x platform will call this to create verticles which it deploys.

It's up to your verticle factory to instantiate the verticle, e.g. by loading script files from the module and executing them.

# Create an API shim that wraps the Vert.x Java API

As a general rule, we don't like to expose the Java API directly to code in the language you are writing the module for.

Language specific APIs should use the idioms common in the language in question. For example, in Vert.x Java we use an interface to represent an event handler, but in Vert.x Groovy we use a Closure, and in Ruby we use a block.

For this reason, you should write a thin API shim in the language in question to convert the raw Java API to a form that is more fitting for the language. You should then include that API shim in the language module.

It's also a general rule of Vert.x language implementations that the API shims should take a similar form across all languages. This means a developer can easily pick up another language API after becoming familiar with one.

# Mark the module as `resident`

Language implementation modules should be marked as "resident": true in `mod.json`. This means they will not get unloaded until the JVM exits.

# Mark the module as `system`

Language implementation modules should be marked as "system": true in `mod.json`. This means they will be installed into the `sys-mods` directory of the Vert.x installation directory and won't be downloaded for every app that needs them

# Tell Vert.x about your new language implementation

Edit `langs.properties` in the Vert.x installation `conf` directory and add a line for your module, e.g.

    cobol=com.company~lang-cobol~1.0.0:com.mycompany.langmod.CobolVerticleFactory

The part to the left of the `=` is the name you give to your language implementation.

The part to the right of the `=` consists of two parts:

The part to the left of the `:` is the module identifier of the module that you wrote. This can be in any repository that Vert.x understands (or it can just be installed locally).

The part to the right of the `:` is the fully qualified class name of your `VerticleFactory` instance that you wrote and that lives inside your module.

If you want to map particular file extensions to your language implementation you can also add a line like:

    .cbl=cobol

This tells Vert.x to use use your module when running any files with the `.cbl` extension.

You can also prefix your main with `cobol:` to force it to use your language module for a particular main, or if you prefer not to make a file extension mapping.

# Push the language implementation to a repository

E.g. Maven Central or Bintray (or oss.sonatype for snapshots)

# Advertise the module in the module registry

Submit the module to the Vert.x [module registry](http://modulereg.vertx.io)

# Next steps

None! That's it.

