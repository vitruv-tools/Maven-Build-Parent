# Vitruv-Maven-Build-Parent
[![GitHub Action CI](https://github.com/vitruv-tools/Maven-Build-Parent/workflows/CI/badge.svg)](https://github.com/vitruv-tools/Maven-Build-Parent/actions?query=workflow%3ACI)
[![Latest Release](https://img.shields.io/github/release/vitruv-tools/Maven-Build-Parent.svg)](https://github.com/vitruv-tools/Maven-Build-Parent/releases/latest)
[![Issues](https://img.shields.io/github/issues/vitruv-tools/Maven-Build-Parent.svg)](https://github.com/vitruv-tools/Maven-Build-Parent/issues)
[![License](https://img.shields.io/github/license/vitruv-tools/Maven-Build-Parent.svg)](https://raw.githubusercontent.com/vitruv-tools/Maven-Build-Parent/main/LICENSE)

A Maven parent specification for all Vitruv-related projects.
It defines default plugin configurations for running the Maven compiler, running Tycho, generating sources, packaging bundles and features, and running tests and for running Xtend.

## Usage

The POM enables all compiler, tycho, packaging, feature, test, bundles and Xtend plugins used in Vitruv. Inheriting from this POM directly enables those plugins in the child projects.

### Test Projects

Projects ending with `.tests` are automatically interpreted as plugin test projects by Maven. The provided POM defines that these tests are executed as pure Jave tests (using the `maven-surefire-plugin` by default.
For cases in which tests need to be run in a full Eclipse workbench based on an Equinox environment, the `tycho-surefire-plugin` with UI harness has to used. To do so, the POM provides a profile that can be activated for a project by placing the marker file `.tests-need-workbench` in the project's root folder.

### MWE2 Workflows

The POM preconfigures the automatic execution of MWE2 workflow executions placed in projects under `workflow/clean.mwe2` for cleanup purposes and under `workflow/generate.mwe2` for generation purposes. The Maven build automatically replaces the variable `workspaceRoot` with the proper path to the workspace root during the build process. So every `mwe2` file should use that variable to define the workspace root for execution within Eclipse (usually relative to the project folder) to be then automatically adjusted during a Maven build.

## Deployment

### Automatic

Deployment is automatically performed by GitHub Actions. Snapshot versions committed to the `develop` branch are deployed to the Sonatype Snapshots repository. For release versions, the according commit on the `main` branch has to be tagged with the version number and a [GitHub release](https://github.com/vitruv-tools/Maven-Build-Parent/releases) has to be created, which triggers deployment to the Sonatype Release repository.

Both the Sonatype user and the GPG key used for signing are provided by secrets of the GitHub repository.

### Manual

Follow the [Sonatype documentation](https://central.sonatype.org/pages/apache-maven.html) for deployment instructions.

Having set up a GPG key, use `mvn clean deploy` with an appropriate Sonatype Jira user defined in your `settings.xml` to deploy a snapshot to Sonatype.

To make a release, first change the version to a release version and then run `mvn clean deploy -P release`. This stages the release on the Sonatype Nexus server. To confirm the release and start the sync to Maven Central, run `mvn nexus-staging:release -P release`.
Alternatively, confirm the release in the staging repository at the [Sonatype Nexus repository](https://oss.sonatype.org/).

