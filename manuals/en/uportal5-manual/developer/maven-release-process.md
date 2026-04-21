# Releasing Components in the uPortal Ecosystem via Maven

## Prerequisites

There are 3 prerequisites to cutting maven releases:

1.  [Sonatype Account at Central Publisher Portal](https://central.sonatype.com/)
    -   NOTE!! Do not link a social account — create a local account.
    -   See the first part of [Register to Publish Via the Central Portal](https://central.sonatype.org/register/central-portal/).
    -   **Note:** The legacy OSSRH service (`oss.sonatype.org`) and the Sonatype JIRA account-creation flow were sunset in June 2025. All publishing — and all namespace-permissions requests — now go through the Central Publisher Portal.
2.  Permissions at Sonatype to release projects
    -   Namespace permissions are managed via the [Central Publisher Portal](https://central.sonatype.com/).
    -   If you need access to the `org.jasig.portlet` (or another uPortal ecosystem) namespace, contact a uPortal committer.
    -   Expect approval to take a few days to complete.
3.  [Set up public PGP key on a server](https://central.sonatype.org/pages/working-with-pgp-signatures.html)
    -   Generate a key pair `gpg2 --gen-key`
    -   If you choose to have an expiration date, edit the key via `gpg2 --edit-key {key ID}`
    -   Determine the key ID and keyring file `gpg2 --list-keys` (the key ID is the `pub` ID)
    -   Distribute your public key `gpg2 --keyserver hkp://keyserver.ubuntu.com --send-keys {key ID}`
        -   The `pool.sks-keyservers.net` pool was decommissioned in 2021 — do not use it.

## Setup

Export your secret keyring via `gpg2 --keyring secring.gpg --export-secret-keys > ~/.gnupg/secring.gpg`

Create a Central Publisher Portal user token via <https://central.sonatype.com> > Profile > Generate User Token. Tokens are required — login password auth is not supported.

In `$HOME/.m2/settings.xml`, configure Maven with your Central Portal user token. The server `<id>` must match the `<distributionManagement>` repository id used by the component you are releasing. For components that still inherit the legacy `<id>sonatype-nexus-staging</id>` (e.g. older portlets), include that id too; for projects migrated to the Central Portal, use `<id>central</id>`:

```xml
<settings>
    <servers>
        <server>
            <id>central</id>
            <username>user_token_name</username>
            <password>user_token_pw</password>
        </server>
        <server>
            <id>sonatype-nexus-staging</id>
            <username>user_token_name</username>
            <password>user_token_pw</password>
        </server>
    </servers>
</settings>
```

Setup is only required to be done once.

### Verify distribution URLs before every release

The legacy OSSRH host `oss.sonatype.org` was sunset in June 2025. Every `<distributionManagement>` URL in the component's POMs — and any inherited from parent POMs (notably `uportal-portlet-parent` and `org.sonatype.oss:oss-parent`) — must point at the Central Portal's OSSRH Staging API compatibility service.

**Required URLs:**

-   Release: `https://ossrh-staging-api.central.sonatype.com/service/local/staging/deploy/maven2/`
-   Snapshot: `https://central.sonatype.com/repository/maven-snapshots/`

**Audit before releasing** (from the component root):

```sh
grep -RIn --include=pom.xml -E 'oss\.sonatype\.org|sonatype-nexus-staging' .
```

If any stale OSSRH URLs appear, override them with a `<distributionManagement>` block in the component's top-level `pom.xml` so the inherited values are shadowed. Long-term, release an updated parent POM with the new URLs so every descendant picks them up automatically.

## Which Repo?

We encourage performing releases directly from a clone of the official repository rather than a fork to avoid extra steps.

This means when testing on `uPortal-start` for the release, you should use the `apereo` repository but configure the version of the component (eg the Bookmarks or Announcements portlet) to be the `SNAPSHOT` version that you'll build in the following steps.

## Send Pre-Release Notice to Community

For any non-snapshot release, email a notice to [`uportal-dev`](https://groups.google.com/a/apereo.org/forum/#!forum/uportal-dev) a couple of working days prior to cutting the release.

-   Request any in-progress Pull Requests be merged by a certain date/time.
-   Request deployers test the tip of `master`.

## Testing

Build a clean version of `uPortal-start` with the quickstart data set and perform at least light testing, especially around features that have been fixed or enhanced.

### Prepare For Testing

Check the styling:
```sh
$ mvn notice:generate
$ mvn license:format
$ mvn javadoc:fix
```

Review any changes from the styling tasks and commit them.

If you run into issues with the Maven task not finding licenses for dependencies, Maven will create a dependency mapping file that you should fold into the primary `license-mapping.xml` file located at <https://github.com/Jasig/apereo-parent/blob/master/licenses/license-mappings.xml>.

Install the build locally:
```sh
$ mvn clean install
```

Ideally, committers will be running the `mvn` `notice`, `license`, and `javadoc` before they commit changes to the repo.  If there are changes needed, review the differences, and then commit to `master`:
```sh
$ git commit -am "chore: pre-release prep"
```

Point your uPortal-start to the local component build:
Using the component's version, configure uPortal-start to pick it up in `.../apereo/uPortal-start/gradle.properties`.

For example, if you are building version `3.5.1-SNAPSHOT` of the Bookmarks portlet, you'd configure `.../apereo/uPortal-start/gradle.properties` with `bookmarksPortletVersion=3.5.1-SNAPSHOT`.

Run uPortal-start with the local build
```sh
$ ./gradlew clean portalInit
```

### Verify

*   Unit tests are automatically run on commits, so ensure the latest commit's CI build passed.
*   FUTURE - Need to run the cross browser platform tests and the performance tests
*   Smoke test the UI manually of the component
    *   Pay attention to new changes

## Cut Release

If you had to commit changes for the styling, push them to the repo.

Run the following command in the directory of the component's clone.  If this is your first time, run each `release:` command separately so if there's a failure in the process, it'll be easier to diagnose and restart.

```sh
$ mvn release:clean release:prepare release:perform
```

:Note: During the `release` tasks, you will be prompted for a release version (e.g. `3.5.1`), release tag, and new development version.  Press `Enter` for the defaults.  The release process will then create two commits - a commit to set the new version (`3.5.1`), and a commit to set the version to the new snapshot (`3.5.2-SNAPSHOT`)

## Verify and Publish from Central Publisher Portal

Because Maven's `deploy` phase uses the legacy Nexus 2-flavored upload path, the OSSRH Staging API does **not** automatically close the staging repository after `release:perform` uploads the artifacts. You must manually trigger the upload into the Central Portal and then publish from the portal UI.

### Push staged artifacts to the portal

Run the following from the **same machine** that ran `release:perform` — the Central Portal matches staging repositories by source IP:

```sh
$ curl -X POST \
    "https://ossrh-staging-api.central.sonatype.com/manual/upload/defaultRepository/{groupId}" \
    -H "Authorization: Bearer $(echo -n '{username}:{password}' | base64)"
```

Replace `{groupId}` with the component's root namespace (e.g. `org.jasig.portlet`, `org.jasig.resourceserver`) and `{username}`/`{password}` with your Central Portal user token credentials (the same ones in `~/.m2/settings.xml`).

If the deployment does not appear in the portal UI, search for open staging repositories:

```sh
$ curl -X GET \
    "https://ossrh-staging-api.central.sonatype.com/manual/search/repositories?ip=any&profile_id={groupId}" \
    -H "Authorization: Bearer $(echo -n '{username}:{password}' | base64)"
```

### Review and publish

1.  Log into <https://central.sonatype.com>
2.  Navigate to **Deployments** — the staged artifacts should now be visible.
3.  Review the release for the expected artifacts (parent POM + every submodule, including any WAR modules).
4.  Verify the artifacts pass validation checks (signatures, POM requirements, matching `<packaging>` — see below).
5.  Publish the deployment to make it available on Maven Central.

### A note on POM packaging

Central Portal validation rejects deployments where the advertised `<packaging>` does not match the uploaded artifact's file extension. This bit uPortal during the 2026 release: a Gradle `uploadArchives` block hardcoded `packaging 'jar'` for every submodule, which broke the WAR module. In Maven, `<packaging>` is authoritative per-module, so the bug does not occur the same way — but confirm each module declares the right packaging:

```sh
$ grep -l '<packaging>war</packaging>' $(find . -name pom.xml -not -path '*/target/*')
```

## Create Release Notes

1.  Use Git to inspect the incremental commits since the last release (e.g. `$ git log v3.5.0..3.5.1 --no-merges`)
2.  Review the issue tracker and confirm that the referenced issues have been Resolved
3.  Enter the release notes on the GitHub releases page in the component's repo
    -   It's helpful to use the previous release notes as a guide
    -   Each commit type goes into a sub section with the type as a header (Fixes, Chores, Features, etc...).

## Update uPortal-start


Open a Pull Request on `uPortal-start` to update the version of the newly released component.  Do a quick smoke test to ensure it built:

```sh
$ cd {uPortal-start repo}
$ ./gradlew tomcatStop
.../uPortal-start$ ./gradlew clean tomcatDeploy
.../uPortal-start$ ./gradlew tomcatStart
```

## Publish new docker demo of Quickstart

Publish a new apereo/uPortal-demo Docker image and update the `:latest` tag.
Prerequisites:
-   Docker Cloud account (<https://cloud.docker.com>)
-   Access to post to the apereo Docker group
-   The uPortal version has been added to `uPortal-start`

```sh
$ cd {uPortal-start repo}
$ ./gradlew dockerBuildImageDemo
$ docker login
$ docker tag {version in format like 5.1.0} apereo/uportal-demo:latest
$ docker push apereo/uportal-demo:latest
```

## Update Community

For any non-snapshot release, email an announcement to [`uportal-dev`](https://groups.google.com/a/apereo.org/forum/#!forum/uportal-dev) and [`uportal-user`](https://groups.google.com/a/apereo.org/forum/#!forum/uportal-user).

-   Be sure to acknowledge those who contributed to the release.
-   Be sure to put the release into context for existing deployers to understand.

Have someone with access to the [uPortal Twitter account](https://twitter.com/uportal) announce the release.

## References

-   <https://apereo.atlassian.net/wiki/spaces/UPC/pages/102336655/Cutting+a+uPortal+Release>
-   <https://central.sonatype.org/publish/publish-portal-guide/>
-   <https://central.sonatype.org/publish/publish-portal-ossrh-staging-api/>
-   <https://maven.apache.org/maven-release/maven-release-plugin/>
-   <https://docs.gradle.org/current/userguide/signing_plugin.html#sec:signatory_credentials>

