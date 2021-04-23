# Vitruv-Maven-Build-Parent
A Maven parent specification for all Vitruv-related projects.
It defines default plugin configurations for running the Maven compiler, running Tycho, generating sources, packaging bundles and features, and running tests and for running Xtend.

## Usage

The POM enables all compiler, tycho, packaging, feature, test, bundles and Xtend plugins used in Vitruv. Inheriting from this POM directly enables those plugins in the child projects.

### Test Projects

Projects ending with `.tests` are automatically interpreted as plugin test projects by Maven. By default, they set up an Eclipse/Equinox platform to run the tests on.
If the test can be run without such a platform, it improves performance when running them as ordinary JUnit test. To do so, the POM provides a profile that can be activated for a project by placing the marker file `.tests-without-platform` in the project's root folder.
In addition, some needs to be run in a full Eclipse workbench, i.e., with UI harness. To do so, the POM provides a profile that can be activated for a a project by placing the marker file `.tests-need-workbench` in the project's root folder.
.tests-without-platform

### Aggregated Updatesites

For building aggregated updatesites with the CBI aggregator, the aggregator file named `updatesite.aggr` has to be placed within the project's main folder to enable building the updatesite.

## Deployment

### Automatic

Deployment is automatically performed by GitHub Actions. Snapshot versions committed to the `master` branch are deployed to the Sonatype Snapshots repository. Commits with release versions that are also tagged as a release are deployed to the Sonatype Release repository.

Both the Sonatype user and the GPG key used for signing are provided by secrets of the GitHub repository.

### Manual

Follow the [Sonatype documentation](https://central.sonatype.org/pages/apache-maven.html) for deployment instructions.

Having set up a GPG key, use `mvn clean deploy` with an appropriate Sonatype Jira user defined in your `settings.xml` to deploy a snapshot to Sonatype.

To make a release, first change the version to a release version and then run `mvn clean deploy -P release`. This stages the release on the Sonatype Nexus server. To confirm the release and start the sync to Maven Central, run `mvn nexus-staging:release -P release`.
Alternatively, confirm the release in the staging repository at the [Sonatype Nexus repository](https://oss.sonatype.org/).

