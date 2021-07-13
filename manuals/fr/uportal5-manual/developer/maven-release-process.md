# Releasing Components in the uPortal Ecosystem via Maven

## Prerequisites

There are 3 prerequisites to cutting maven releases:

1. [JIRA Account at Sonatype](https://issues.sonatype.org/secure/Signup!default.jspa)
2. Permissions at Sonatype to release projects
    - This is granted via a Jira ticket from a uPortal committer
3. [Set up public PGP key on a server](https://central.sonatype.org/pages/working-with-pgp-signatures.html)
    - Generate a key pair `gpg2 --gen-key`
    - If you choose to have an expiration date, edit the key via `gpg2 --edit-key {key ID}`
    - Determine the key ID and keyring file `gpg2 --list-keys` (the key ID is the `pub` ID)
    - Distribute your public key `gpg2 --keyserver hkp://pool.sks-keyservers.net --send-keys {key ID}`

## Setup

Export your secret keyring via `gpg2 --keyring secring.gpg --export-secret-keys > ~/.gnupg/secring.gpg`

Create a Sonatype user token via <https://oss.sonatype.org> > Profile > User Tokens

In `$HOME/.m2/settings.xml`, configure Maven with your Sonatype user token:

```xml
<settings>
    <servers>
        <server>
            <id>sonatype-nexus-staging</id>
            <username>user_token_name</username>
            <password>user_token_pw</password>
        </server>
    </servers>
</settings>
```

Setup is only required to be done once.

## Which Repo?

We encourage performing releases directly from a clone of the official repository rather than a fork to avoid extra steps.

This means when testing on `uPortal-start` for the release, you should use the `apereo` repository but configure the version of the component (eg the Bookmarks or Announcements portlet) to be the `SNAPSHOT` version that you'll build in the following steps.

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

* Unit tests are automatically run on commits, so ensure the latest commit's CI build passed.
* FUTURE - Need to run the cross browser platform tests and the performance tests
* Smoke test the UI manually of the component
  * Pay attention to new changes

## Cut Release

If you had to commit changes for the styling, push them to the repo.

Run the following command in the directory of the component's clone.  If this is your first time, run each `release:` command separately so if there's a failure in the process, it'll be easier to diagnose and restart.

```sh
$ mvn release:clean release:prepare release:perform
```

:Note: During the `release` tasks, you will be prompted for a release version (e.g. `3.5.1`), release tag, and new development version.  Press `Enter` for the defaults.  The release process will then create two commits - a commit to set the new version (`3.5.1`), and a commit to set the version to the new snapshot (`3.5.2-SNAPSHOT`)

## Close and Release from Nexus Staging Repository

Close the release in Sonatype to ensure the pushed Maven artifacts pass checks:
1. Log into <https://oss.sonatype.org>
2. Search among `Build Promotion` > `Staging Repositories` for your username
3. Review the release for the expected artifacts
4. Select the uPortal artifact you staged and hit "Close"
  - The lifecycle is `Open --> Closed --> Released`
5. Wait a few minutes for the component's artifact to close
6. Release the artifact, and put the tag name in the description
  - Leave 'Automatically Drop' checked.

## Create Release Notes

1. Use Git to inspect the incremental commits since the last release (e.g. `$ git log v3.5.0..3.5.1 --no-merges`)
2. Review the issue tracker and confirm that the referenced issues have been Resolved
3. Enter the release notes on the GitHub releases page in the component's repo
  - It's helpful to use the previous release notes as a guide
  - Each commit type goes into a sub section with the type as a header (Fixes, Chores, Features, etc...).

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
  - Docker Cloud account (<https://cloud.docker.com>)
  - Access to post to the apereo Docker group
  - The uPortal version has been added to `uPortal-start`

```sh
$ cd {uPortal-start repo}
$ ./gradlew dockerBuildImageDemo
$ docker login
$ docker tag {version in format like 5.1.0} apereo/uportal-demo:latest
$ docker push apereo/uportal-demo:latest
```

## Update Community
For any non-snapshot release, email an announcement to [`uportal-dev`](https://groups.google.com/a/apereo.org/forum/#!forum/uportal-dev) and [`uportal-user`](https://groups.google.com/a/apereo.org/forum/#!forum/uportal-user).
  - Be sure to acknowledge those who contributed to the release.
  - Be sure to put the release into context for existing adopters to understand.

Have someone with access to the [uPortal Twitter account](https://twitter.com/uportal) announce the release.

## References

(<https://apereo.atlassian.net/wiki/spaces/UPC/pages/102336655/Cutting+a+uPortal+Release>)
(<https://central.sonatype.org/pages/ossrh-guide.html>)
(<https://docs.gradle.org/current/userguide/signing_plugin.html#sec:signatory_credentials>)

