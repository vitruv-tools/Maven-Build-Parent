# Vitruv-Maven-Build-Parent
A Maven parent specification for all Vitruv-related projects.
It defines default plugin configurations for running the Maven compiler, running Tycho, generating sources, packaging bundles and features and running tests and for running Xtend.

## Usage

You can put the following marker files in the projects with this parent to disable certain features:

`.maven_disable_default-plugin-configurations`: Disables the default plugin configurations for Maven compilter, Tycho, sources, packaging, tests etc. (only disables plugin configuration, not their usage)

`.maven_disable_default-plugin-usage`: Disables the default plugin usage for the default configurations (only disables plugin usage, not their configuration)

`.maven_disable_compile-xtend`: Disables the Xtend compiler

## Deployment

Follow the [Sonatype documentation](https://central.sonatype.org/pages/apache-maven.html) for deployment instructions.

Having set up a PGP key, use `mvn clean deploy` with an appropriate Sonatype Jira user defined in your settings.xml to deploy a snapshot to Sonatype.

To make a release, first change the version to a release version and then run `mvn clean deploy -P release`. This stages the release on the Sonatype Nexus server. To confirm the release and start the sync to Maven Central, run `mvn nexus-staging:release`

