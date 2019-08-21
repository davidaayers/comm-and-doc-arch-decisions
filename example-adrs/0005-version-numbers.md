# 5. Version Number Conventions

Date: 09/27/2017

## Context

We manage many different versions of applications, services and libraries across multiple environments.

As we think about how we want to identify our artifacts with version numbers, we have the following goals:

* As much as possible we want to follow existing industry conventions so that
  * it's easier for new developers to understand
  * it's easier for external parties to adopt and understand any APIs we expose
  * we can take advantage of any tooling that relies upon those conventions
* We want our "full" version number to uniquely identify an artifact.
  If a commit is built a second time, it should have a different version number because
  the resulting artifact may have different bits.
* We want to be able to easily identify the latest artifact,
  i.e. version numbers should have a chronological order.
* Every build should create a potentially releasable artifact.
  That is to say, we shouldn't need to change anything about the artifact to deploy it to production,
  which includes not changing the version number that is embedded in the artifact.
* We'd like to be able to easily identify which artifacts are still a work-in-progress
  vs which are "released" (have passed QA and are deemed stable).
  * Marking an artifact as "released" should not change its contents in any way.
    i.e. It should be exactly what QA tested.
* We'd like to have a "release" version number (e.g. 1.2.3) that we can anticipate in advance
  to identify a release for use in planning, Pivotal labels, HEAT, etc.
* Backward-incompatible changes should be easy to identify
  so that dependent projects don't pull in breaking changes by accident.
* For the purposes of capitalizing our time, we want to be able to distinguish between new feature work
  and bug fixes to existing releases.
  From an accounting standpoint, there are some versions that will be considered assets to be depreciated
  and others that will be treated as expenses.
* We would like to be able to easily trace back to the commit from which an artifact was built.

It's also helpful to keep in mind that dependencies between our components take a couple of different forms:

* End-user applications that have no other components that depend on them.
  Therefore there's really no such thing as a backward-incompatible change.
* Services that expose APIs provide their updates automatically to consumers.
  That is to say that dependent components don't need to change anything in order to get the latest bug fixes.
  That being said, backward incompatible changes (i.e. new major versions) require establishing a new endpoint
  so that dependents are forced to make an explicit change to consume the breaking changes.
* For shared libraries that are compile-time dependencies,
  the dependent components need to rebuild to consume the new changes.
  Usually the dependent components will need to make a change to their dependency reference
  (e.g. change the maven dependency coordinates from 1.2.3 to 1.2.4).

Even though the uses of version numbers are a little dependent on the type of component being produced,
we would like our version numbers to be as consistent as possible to reduce confusion.
Database migrations are an exception since they have a very different set of concerns.

## Decision

We will use [Semantic Versioning](http://semver.org/) as a basis for all of our applications, services and libraries,
with some TCS-specific interpretations.

Given a version number MAJOR.MINOR.PATCH-BUILD-COMMIT:

* Increment the MAJOR version when you make incompatible API changes.
  For applications that have no dependencies,
  increment the major version when the team deems that it is a significant enough change, but this is quite rare.
* Increment the MINOR version when you add functionality in a backwards-compatible manner.
* Increment the PATCH version when you make backwards-compatible bug fixes.
  The PATCH version number should only be incremented once for each release
  so that the MAJOR.MINOR.PATCH can be known in advance for release planning purposes.
* Increment the BUILD version when you build the artifact.
  This doesn't need to start at 0, and it's ok if it has gaps (e.g. jump from 23 to 26),
  but it does need to increase with each subsequent build within a MAJOR.MINOR.PATCH.
* Include the 7-character git COMMIT sha.
  Because this comes after the build version, the commit won't be material when ordering versions
  or even for uniquely identifying the version (the previous four numbers can serve as a unique identifier),
  but it serves as a useful piece of metadata for jumping right to the last commit included in the build.  

#### Where versions are recorded

* MAJOR, MINOR and PATCH are under the developer's control and should be tracked in version control.
* BUILD and COMMIT are added by the build system and should not be tracked in version control.

Some frameworks have an established place to record the version information
(e.g. Maven `pom.xml`, RubyGems `.gemspec`, Node `package.json`) and we should follow those conventions.

In cases where we are developing software and there is no established standard for where to store the 
version information, we will create a `.version` file in the application's root directory that will be tracked in git.

The contents of the file will be one line of the form
```
[MAJOR].[MINOR].[PATCH]
```

In some cases (e.g. Gradle) the contents of the `.version` file may include a `-SNAPSHOT` suffix, e.g.
```
1.2.4-SNAPSHOT
```
but that `-SNAPSHOT` suffix should be ignored when extracting the MAJOR.MINOR.PATCH.

When the CI server builds the application, it will create a `build.properties` file
in the application's root directory that includes the full version (with build number and commit)
in a line like
```
build.version=3.38.2-4-ace34d5
```

The build.properties file will only exist in the artifact and will not be committed to git.

#### Releasing

After an artifact has QA's approval, it can be "released."

This means that it is the most stable and final artifact for that MAJOR.MINOR.PATCH. 

Releasing an artifact generally involves:
* Identifying the appropriate MAJOR.MINOR.PATCH release label
* Tagging the commit in git with that release label
* Incrementing the PATCH version in git so that no other builds will use this same MAJOR.MINOR.PATCH 
* Publishing the artifact wherever appropriate under that release label.
  Note that in the Java world with Maven dependencies, the artifact will need to be published in Nexus
  under the full version number, not an abbreviated release label. 
  

#### A note on semver and our build numbers

The semver specification refers to [build information being appended with + sign](http://semver.org/#spec-item-10),
however the specification also states that "Build metadata SHOULD be ignored when determining version precedence."
Since our build numbers do serve as a meaningful indicator of precedence, we will not use the `+` suffix for them.

The specification does define a precedence for
[pre-release versions that are denoted by a hyphen](http://semver.org/#spec-item-9),
so we have chosen to put our build number after the `-`.


## Consequences

A standard approach to formatting and recording version information will help developers
who are new to a project get their bearings, as well as provide protection against breaking changes in dependencies.
It will also provide opportunities to leverage common scripts and tooling
across more projects.

For determining capitalizable assets, we can assume that any version with a `.0` PATCH is new
functionality, whereas any PATCH > 0 are bug fixes and not capitalizable.

Our releases will generally be known by their MAJOR.MINOR.PATCH label, which references a specific
full version (MAJOR.MINOR.PATCH-BUILD-COMMIT) that was blessed by QA.

## Related Categories

* SDLC
