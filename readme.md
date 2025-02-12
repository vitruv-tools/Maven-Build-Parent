# Vitruvius Maven Build Parent

*A common parent POM for all builds of vitruv.tools*

## Usage

Set the build parent POM as parent in your project POM.
Remember to replace `x.y.z` with the actual version you want to reference.

```
<!-- Build Parent -->
<parent>
    <groupId>tools.vitruv</groupId>
    <artifactId>parent</artifactId>
    <version>x.y.z</version>
</parent>
```

If you want to use a snapshot version of the Maven build parent (or of any other dependency), you need to enable snapshot dependencies from the OSSRH repository as shown below.

```
<repositories>
    <!-- allow snapshots -->
    <repository>
      <id>ossrh-snapshots</id>
      <name>OSSRH Snapshots</name>
      <url>https://oss.sonatype.org/content/repositories/snapshots</url>
      <snapshots>
        <enabled>true</enabled>
      </snapshots>
      <releases>
        <enabled>false</enabled>
      </releases>
    </repository>
</repositories>
```

An example project showcasing the usage and capabilities of the Maven build parent is the [EMF-Template](https://github.com/vitruv-tools/EMF-Template) project in the [vitruv-tools](https://vitruv.tools/) GitHub organization.

## Requirements

To generate Java code from Xtend code, you should use a JDK with the same version as the source/target compiler version in your Maven build, as otherwise the xtend-maven-plugin could generate code that uses parts of the Java API that are not available in the used JDK.
Currently the parent POM in this project specifies Java 17 as source/target compiler version, so you should use a JDK with version 17 for the build.

## Provided Plugins

### Build Infrastructure

#### `org.codehaus.mojo:build-helper-maven-plugin`

- required for adding code in non-Java languages (such as Xtend), as well as generated code (for Xtend and Ecore) to the classpath
- the `xtend-maven-plugin` relies on Xtend files being available on the classpath
- the `maven-compiler-plugin` relies on generated code being available on the classpath

#### `org.codehaus.mojo:exec-maven-plugin`

- the default phase `execute-mwe2-generate` is used to execute an MWE2 workflow which, e.g., generates Java code from Ecore meta-models or Xtext grammars
- if used, the workflow file is required to be named `generate.mwe2` and placed in the top-level directory `workflow`
- the workflow should place generated files in `${project.build.directory}/generated-sources`

#### `org.eclipse.xtext:xtext-maven-plugin`

- the execution phase `generate-sources` is used to generate Java or Xtend code from code written in a Vitruvius DSL
- if Xtend code is referenced from the DSL and vice versa, the Xtend code needs to be processed by the xtext-maven-plugin as well, in this case the `xtend-maven-plugin` should be disabled
- projects using the plugin need to configure the DSL (see below)

#### `org.eclipse.xtend:xtend-maven-plugin`

- used to generate Java code from (production and test) Xtend code
- the generated code is placed in `${project.build.directory}/generated-sources/xtend` or `${project.build.directory}/generated-test-sources/xtend`

### Code Analysis

#### `org.jacoco:jacoco-maven-plugin`

- calculates the code coverage from the unit tests for the individual modules

#### `org.sonarsource.scanner.maven:sonar-maven-plugin`

- starts the SonarQube cloud analysis
- projects need to specify the following information as Maven properties

```
<sonar.host.url>https://sonarcloud.io</sonar.host.url>
<sonar.organization>vitruv-tools</sonar.organization> <!-- replace with your organization ID, if necessary -->
<sonar.projectKey>vitruv-tools_XYZ</sonar.projectKey> <!-- replace with your project key -->
```

### Deployment

#### `maven-jar-plugin`

- packages the build results in a jar archive
- should be disabled for test-only projects by setting an empty `phase` for the execution `default-jar`
- when packaging Ecore meta-models, the `MANIFEST.MF` file needs to be added to the jar, which is achieved by adding the following code to the plugin configuration

```
<archive>
    <manifestFile>${project.basedir}/META-INF/MANIFEST.MF</manifestFile>
</archive>
```

#### `org.codehaus.mojo:flatten-maven-plugin`

- replaces the POM of a project with an effective, flattened POM
- configured to remove references to external repositories
- can be used for dependency wrapper modules
- removes empty tags, so be careful with tags that are required for deployment, e.g., `description`

#### `maven-shade-plugin`

- includes dependencies in the built jar archive
- removes references to dependencies from the POM in the built jar archive
- can be used for dependency wrapper modules

#### `maven-source-plugin`

- creates a JAR containing the source files of the module

#### `maven-javadoc-plugin`

- creates a JAR containing the JavaDoc files of the module
- contains the necessary configuration for EMF JavaDoc tags

#### `maven-gpg-plugin`

- signs generated jar archives for deployment
- automatically enabled by the Maven profile `snapshot`

#### `maven-install-plugin`

- copies the packaged build results in the users local repository
- must be disabled if `maven-jar-plugin` is disabled

## Provided Profiles

Profiles combine plugin activations for different use cases.
The Maven build parent currently supports the following profiles, which can be activated using the flag `-P <name>` where `<name>` is the name of the profile.

### Profile `coverage`

activates the code coverage analysis

| Activated Plugins                |
|----------------------------------|
| `org.jacoco:jacoco-maven-plugin` |

### Profile `snapshot`

used for snapshot deployment

| Activated Plugins  |
|--------------------|
| `maven-gpg-plugin` |

### Profile `release`

used for release deployment

| Activated Plugins                                 |
|---------------------------------------------------|
| `maven-source-plugin`                             |
| `maven-javadoc-plugin`                            |
| `maven-gpg-plugin`                                |
| `org.sonatype.plugins:nexus-staging-maven-plugin` |

## Expected Project Structure

The expected project structure is an extension of the standard Maven project structure.
Not all of the shown directories and files are required for every project, use as applicable.
The `reactions` directory is an example for a directory, in which code written in a DSL supported by Vitruvius should be placed.

The file `.project` is only required when code is generated from Ecore meta-models or Xtext grammars.
The file `plugin.xml` is automatically generated when working with Ecore meta-models and will contain generated extension point definitions.
It can be modified to include additional extension point definitions, e.g., when using custom factories for testing purposes.

```
src/
- main/
    - ecore/
    - java/
    - reactions/
    - xtend/
    - xtext/
    - resources/
- test/
    - java/
    - xtend/
    - resources/
workflow/
    - generate.mwe2
.project
plugin.xml
pom.xml
```

### Examples

The folder `examples/` contains examples of files required for the build process.
The following list described the purpose of the files, as well as the necessary changes for using them.

- `.project` - marks the root directory
    - Change the name (`my.project.model`) to the model plugin ID (usually the Maven artifact ID).
- `generate.mwe2` - defines the workflow for building Ecore meta-models and Xtext grammars
    - Change the module name (`my.project.model`) to the model plugin ID (usually the Maven artifact ID).
    - Add/remove components, as required for the project (one `EcoreGenerator` per Ecore meta-model, one `XtextGenerator` per Xtext language).
    - Change the names and paths as applicable, while following the expected project structure.
    - For `XtextGenerator`s, enable or disable the generator features (e.g., `serializer`, `generator`) as applicable.

### Working with Ecore Meta-Models

The Maven build parent defines a process and a collection of Maven plugins for working with `.ecore` and `.genmodel` files as seamlessly as possible.
It can, however, be the case, that small changes to the files are necessary, mostly due to the way Eclipse handles URIs.
When working with EMF technology, there are four kinds of URIs, which should be used in different contexts (or not at all).
Referencing of elements, e.g., a specific class or attribute, inside an artifact, e.g., a meta-model, is possible with relative paths after an `#` at the end of a URI (see examples below).

| Name                     | Description                                                                                                                                     | Example                                                      |
|--------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------|
| Namespace URIs (NS-URIs) | Global, absolute identifier for artifacts, mainly meta-models                                                                                   | `http://www.eclipse.org/emf/2002/Ecore#//ecore`              |
| Platform Resource URIs   | File-based URI for resources contained in packages. Contains the plugin/artifact ID [1] and a path relative to the package, e.g., inside a JAR. | `platform:/resource/org.eclipse.emf.ecore/model/Ecore.ecore` |
| Other Platform URIs      | Eclipse-specific URIs for resources, must not be used.                                                                                          | `platform:/plugin/...`                                       |
| Relative URIs            | Relative URIs for artifacts, should be avoided.                                                                                                 | `../../model/Ecore.ecore`                                    |

For `.genmodel` files:
- the main `genmodel:GenModel` tag needs to contain the attribute `modelPluginID` with the model plugin ID [1] as its value
- all genpackages in the `usedGenPackages` tag need to be referenced using `platform:/resource/` URIs
    - yes: `platform:/resource/org.eclipse.emf.ecore/model/Ecore.genmodel#//ecore`
    - no: `../../org.eclipse.emf.ecore/model/Ecore.genmodel#//ecore`
    - no: `http://www.eclipse.org/emf/2002/Ecore#//ecore`
- the model directory in the `modelDirectory` tag should be `/<model-plugin-id>/target/generated-sources/ecore`

For `.ecore` files:
- all URIs should be namespace URIs
    - yes: `http://www.eclipse.org/emf/2002/Ecore#//EClassifier`
    - no: `../../org.eclipse.emf.ecore/model/Ecore.ecore#//EClassifier`
    - no: `platform:/resource/org.eclipse.emf.ecore/model/Ecore.ecore#//EClassifier`

Examples for both kinds of files can be found in the EMF-Template repository.
A small [genmodel](https://github.com/vitruv-tools/EMF-Template/blob/7ba1d2a49971e56cdf9e9cac3a5373c70db16777/modelextension/src/main/ecore/extended-model.genmodel) file shows the required tags and the use of `platform:/resource` URIs, while an [ecore](https://github.com/vitruv-tools/EMF-Template/blob/7ba1d2a49971e56cdf9e9cac3a5373c70db16777/modelextension/src/main/ecore/extended-model.ecore) file shows the use of namespace URIs.

Sometimes it can be necessary to explicitly define a mapping between namespace URIs and `platform:/resource/` URIs for referenced meta-models. In this case, add the following to the `StandaloneSetup` definition in the `generate.mwe2` workflow file (replace the URIs as appropriate):

```
uriMap = {
    from = "http://www.eclipse.org/emf/2002/Ecore"
    to = "platform:/resource/org.eclipse.emf.ecore/model/Ecore.ecore"
}
```

[1] The model plugin ID is usually the artifact ID of the Maven project. It must be used consistent in the genmodel, the `.project` file and the `generate.mwe2` workflow file (for the `platform:/resource/` URIs, as well as for the module name at the beginning of the file).

### Working with Vitruvius DSLs

Generating Xtend or Java code from Vitruvius DSLs, like the Reactions language, requires setting up the classpath, as well as configuring the `xtext-maven-plugin` for the used language.

In order for the `xtext-maven-plugin` to find the code written in the DSL, the directory in which the code is placed, as well as the output directory for the generated Xtend or Java code need to be specified.
This is done by adding a source to the configuration of a new execution of the `build-helper-maven-plugin`.
In the example below, replace `reactions` with the name of the used DSL.
It is important not to use the name `add-source` as the `id` of the execution, as this would overwrite the classpath setup done the in this build parent.

```
<plugin>
    <groupId>org.codehaus.mojo</groupId>
        <artifactId>build-helper-maven-plugin</artifactId>
        <executions>
            <execution>
                <id>add-source-reactions</id>
                <phase>generate-sources</phase>
                <goals>
                <goal>add-source</goal>
            </goals>
            <configuration>
                <sources>
                    <source>${project.basedir}/src/main/reactions</source>
                    <source>${project.build.directory}/generated-sources/reactions</source>
                </sources>
            </configuration>
        </execution>
    </executions>
</plugin>
```

To generate code from the DSL code, the used DSL packages need to be included as a dependencies of the `xtext-maven-plugin` plugin.
If the DSL code references meta-models from foreign packages, these need to be included as dependencies for the build plugin as well.

Below you find an example configuration of the build plugin for the Reactions language.

```
<configuration>
    <languages>
        <language>
            <setup>tools.vitruv.dsls.reactions.ReactionsLanguageStandaloneSetup</setup>
            <outputConfigurations>
            <outputConfiguration>
                <outputDirectory>${project.build.directory}/generated-sources/reactions</outputDirectory>
            </outputConfiguration>
            </outputConfigurations>
        </language>
    <languages>
<configuration>
```

If the DSL code references Xtend code and vice versa, the Xtend code needs to be processed by the `xtext-maven-plugin` as well, instead of using the `xtend-maven-plugin`.
For that, Xtend needs to be configured as an Xtext DSL as well, using the following addition to the configuration.

```
<language>
    <setup>org.eclipse.xtend.core.XtendStandaloneSetup</setup>
    <outputConfigurations>
        <outputConfiguration>
            <outputDirectory>${project.build.directory}/generated-sources/xtend</outputDirectory>
        </outputConfiguration>
    </outputConfigurations>
</language>
```

If the tests of the DSL code include Xtend code, you need to use the `xtext-maven-plugin` and the `xtend-maven-plugin` simultaneously.
However, as the main Xtend code is already processed by the `xtext-maven-plugin`, you need to disable the respective execution of the `xtend-maven-plugin`, as shown below.
This way, only the second execution `compile-test-xtend` is executed to process the Xtend test code.

```
<plugin>
    <groupId>org.eclipse.xtend</groupId>
    <artifactId>xtend-maven-plugin</artifactId>
    <executions>
        <execution>
            <id>compile-xtend</id>
            <phase />
        </execution>
    </executions>
</plugin>
```

### Generated Directories and Files

Generated directories and files should in general not be modified or committed, unless specified otherwise in the following list.

- `META-INF/` contains Eclipse-specific project information
- `target/` contains Maven build results
- `build.properties` and `plugin.properties` also contain Eclipse-specific project information
- `plugin.xml` is generated automatically, but *should be committed* and can be altered with custom settings for Ecore meta-models

## Deployment

The Maven build parent supports snapshot and release deployment via OSSRH.
The deployment process defined by the build parent is also used for the deployment of the build parent itself.

### Requirements

Deploying artifacts via OSSRH requires:

1. a Sonatype account
2. publish authorization for your namespace
3. a distributed PGP key.

A [guide to publishing via OSSRH](https://central.sonatype.org/publish/publish-guide/) is available as part of the Sonatype documentation, as well as a [tutorial for creating a PGP key](https://central.sonatype.org/publish/requirements/gpg/).
The account credentials, as well as the PGP secret key, should be stored as GitHub repository secrets, named, e.g.:

| Secret                        | Description                                                  |
|-------------------------------|--------------------------------------------------------------|
| OSSRH_USERNAME                | Username of your OSSRH user token (not your Sonatype login). |
| OSSRH_TOKEN                   | Password of your OSSRH user token (nor your Sonatype login). |
| OSSRH_GPG_SECRET_KEY          | Secret key generated with GPG.                               |
| OSSRH_GPG_SECRET_KEY_PASSWORD | Password for the GPG secret key.                             |

In addition, the project description in your POM needs to contain certain information described in the Sonatype requirements.
While not all of the requirements need to be met for snapshot deployment, you should provide the information presented in the [example POM](https://central.sonatype.org/publish/requirements/#a-complete-example-pom).
For projects in the Vitruvius Tools organization, certain information, like the `organization` and the `developers`, is provided by the Maven build parent.

### Process

Before you start to deploy your project, make sure the project version is set correctly.
For snapshot deployment, this requires the version suffix `-SNAPSHOT` (case sensitive).
To automatically create commits with the release version and the subsequent snapshot version, use the script `.github/prepare-release` in the root directory of the respective repository.

```sh
# switches to a new branch
# creates a first commit with the release version 3.1.7
# creates a second commit with the snapshot version 3.2.0-SNAPSHOT
.github/prepare-release 3.1.7 3.2.0
```

To manually set the version of a Maven project, including all its submodules, use the following command (replace `x.y.z` with your actual version) or the equivalent on your operating system:

```sh
./mvnw versions:set -DnewVersion=x.y.z-SNAPSHOT -DgenerateBackupPoms=false
```

In general, deployment should only be performed on a build server, using, e.g., GitHub actions.
The workflow used to build and deploy the Maven build parent, located in the `.github/workflows` directory, can be used as a starting point for a project-specific workflow.
