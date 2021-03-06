# Gradle release plugin (Git and Subversion) [![Build Status](http://travis-ci.org/ari/gradle-release-plugin.png?branch=master)](http://travis-ci.org/ari/gradle-release-plugin)

Gradle releases made easy. Other release processes make you store your versioning information inside the project; this plugin keeps versions where they belong, in your version control system. We currently support subversion or git. Additional SCM choices are easy to add.

This plugin doesn't try to take over your release management process. Instead it does two simple and clear things:

## 1. Version numbering

Use this plugin to give your project a version number aligned to the branch or tag you are building from. In the case of a normal gradle build, the plugin generates a version name based on the current branch name.

So let's say you are building a project in subversion and your working copy was checked out from https://svn.acme.com/project/trunk. In this case, the gradle property release.projectVersion will be set to 'trunk-SNAPSHOT'.

If you are building code checked out from https://svn.acme.com/project/tags/1.2, release.projectVersion will be '1.2'. And if you are building from https://svn.acme.com/project/branches/big-refactor, the you'll get 'big-refactor-SNAPSHOT'.

Git is even easier. There, you will have the version set to ${tagName} or ${branchName}-SNAPSHOT. No longer will you be chasing around changing the version in build.gradle as you make new branches or tag your code.

At any time you can override the version number on the command line:

     gradle assemble -PreleaseVersion=2.0-SNAPSHOT


## 2. Release tagging

Just run this to have the plugin automatically guess the next available version number, and create a tag in your version control system of choice.

     gradle release

Or this if you want to set the release version yourself.

     gradle release -PreleaseVersion=4.1.2



### Usage

Add the following to your build.gradle file:

```groovy
buildscript {
  repositories {
    mavenCentral()
  }

  dependencies {
    classpath 'au.com.ish.gradle:release:2.0'
  }
}

apply plugin: 'release'
version = release.projectVersion
````

Most of the above is self-explanatory. Include the buildscript section to pull this plugin into your project. Apply the plugin, and set your project version to the version extracted from your version control system. You can pass further properties like this:

````
release {
  failOnSnapshotDependencies = true
  allowLocalModifications = false
  releaseDryRun = false
  scm = 'svn'
  username = 'fred'
  password = 'secret01'
}
````

* failOnSnapshotDependencies: default is true. Will fail the release task if any dependency is currently pointing to a SNAPSHOT
* allowLocalModifications: defaults to false. Will fail the release task if any uncommitted changes remain in your local version control. This prevents you from releasing a build which you cannot later reproduce because you don't have the complete set of source which went into the build.
* releaseDryRun: this skips the commit of the tag to your version control system
* scm: your choices here are 'git' or 'svn' and it defaults to 'svn'
* username: a username for your version control system. This is mostly useful for running releases from a continuous integration server like Jenkins. If you don't pass this, the release plugin will take credentials from any cached on your system or prompt you for them.
* password: a password to match the username
* scmRoot: The path to your version control repository. This is not needed if you have a simple checkout of the trunk path and only one gradle project beneath that. Some people have different layouts and this hint is needed so that the release plugin knows where to make tags and look for branches. Only used for subversion.

## Properties

You can access two properties from this plugin once you have configured it:

  release.projectVersion
    Read only property for getting the version of this project.
    The version is derived from the following rules:
    1. If the releaseVersion property was passed to gradle via the -P command line option, then use that.
    2. If the version control system is currently pointing to a tag, then use a version derived from the name of the tag
    3. Use the name of the branch (or trunk/head) as the version appended with "-SNAPHOT"
  
  release.scmVersion
    Read only property for getting the version from the source control system.
    This will return the svn commit number or the git hash for the current state of the local repository. This value may be useful for putting into the manifest file of a Java project or since it can be more reliable (but not as pretty) as the public-facing version numbering.


## Tasks

Many people will want to call their build task like this to build, test, tag and upload their release artifacts.

    gradle clean test release uploadArchives