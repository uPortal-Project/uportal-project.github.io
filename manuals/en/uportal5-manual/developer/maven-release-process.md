# Releasing Components in the uPortal Ecosystem via Maven

## Prerequisites

There are 3 prerequisites to cutting maven releases:

1. [Sonatype Account at Central Publisher Portal](https://central.sonatype.com/)
   - NOTE!! Do not link a social account — create a local account.
   - See the first part of [Register to Publish Via the Central Portal](https://central.sonatype.org/register/central-portal/).
   - **Note:** The legacy OSSRH service (`oss.sonatype.org`) and the Sonatype JIRA account-creation flow were sunset in June 2025. All publishing — and all namespace-permissions requests — now go through the Central Publisher Portal.
2. Permissions at Sonatype to release projects
   - Namespace permissions are managed via the [Central Publisher Portal](https://central.sonatype.com/).
   - If you need access to the `org.jasig.portlet` (or another uPortal ecosystem) namespace, contact a uPortal committer.
   - Expect approval to take a few days to complete.
3. [Set up public PGP key on a server](https://central.sonatype.org/pages/working-with-pgp-signatures.html)
   - Generate a key pair `gpg2 --gen-key`
   - If you choose to have an expiration date, edit the key via `gpg2 --edit-key {key ID}`
   - Determine the key ID and keyring file `gpg2 --list-keys` (the key ID is the `pub` ID)
   - **Publish your key on `keys.openpgp.org`** — the Central Publisher Portal queries this keyserver **first** when validating signatures. A key that's only on `keyserver.ubuntu.com` or other SKS-network servers will fail signature validation non-deterministically (sometimes the first release in a session works because of caching, then a subsequent one in the same session fails — verified during the 2026 portlet release cycle).
     - `keys.openpgp.org` does not accept SKS-style API uploads. Export your public key with `gpg2 --export --armor {fingerprint} > pubkey.asc`, then paste the contents into the web form at <https://keys.openpgp.org/upload> and confirm via the email link they send to your key's UID address. Identity packets are gated behind email confirmation; without it, the key publishes but searches against your email won't return it.
     - Optionally also push to `keyserver.ubuntu.com` as a redundancy: `gpg2 --keyserver hkp://keyserver.ubuntu.com --send-keys {key ID}`
     - The `pool.sks-keyservers.net` pool was decommissioned in 2021 — do not use it.
   - **Verify your key is reachable on `keys.openpgp.org`** before every release session:
     ```sh
     $ curl -so /dev/null -w "%{http_code}\n" \
         "https://keys.openpgp.org/vks/v1/by-fingerprint/{FINGERPRINT}"
     ```
     `200` = good. `404` = upload first.

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

- Release: `https://ossrh-staging-api.central.sonatype.com/service/local/staging/deploy/maven2/`
- Snapshot: `https://central.sonatype.com/repository/maven-snapshots/`

**Audit before releasing** (from the component root):

```sh
grep -RIn --include=pom.xml -E 'oss\.sonatype\.org|sonatype-nexus-staging' .
```

If any stale OSSRH URLs appear, override them with a `<distributionManagement>` block in the component's top-level `pom.xml` so the inherited values are shadowed. Long-term, release an updated parent POM with the new URLs so every descendant picks them up automatically.

### Verify required POM metadata

The Central Publisher Portal rejects deployments missing certain POM elements. In addition to the URLs above, every published POM must declare a `<developers>` element. Missing developers info returns:

```
HTTP 400
Deployment reached an unexpected status: Failed
pkg:maven/<groupId>/<artifactId>@<version>
- Developers information is missing
```

**Audit before releasing:**

```sh
grep -rl --include=pom.xml '<developers>' . || echo "MISSING: no <developers> block in any POM"
```

For components inheriting from `uportal-portlet-parent`, the block is inherited automatically (version `44+`). For components on older parent versions, on a different parent (e.g. `spring-boot-starter-parent`), or publishing the parent itself, declare the block explicitly in the top-level `pom.xml`:

```xml
<developers>
    <developer>
        <organization>uPortal Developers</organization>
        <organizationUrl>https://github.com/uPortal-Project/uPortal/blob/master/docs/COMMITTERS.md</organizationUrl>
    </developer>
</developers>
```

Pointing at the `COMMITTERS.md` in the main `uPortal` repo keeps this stable across all ecosystem releases; individual contributors come and go, the committers list is the durable reference.

## Which Repo?

We encourage performing releases directly from a clone of the official repository rather than a fork to avoid extra steps.

This means when testing on `uPortal-start` for the release, you should use the `apereo` repository but configure the version of the component (eg the Bookmarks or Announcements portlet) to be the `SNAPSHOT` version that you'll build in the following steps.

## Send Pre-Release Notice to Community

For any non-snapshot release, email a notice to [`uportal-dev`](https://groups.google.com/a/apereo.org/forum/#!forum/uportal-dev) a couple of working days prior to cutting the release.

- Request any in-progress Pull Requests be merged by a certain date/time.
- Request deployers test the tip of `master`.

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

Ideally, committers will be running the `mvn` `notice`, `license`, and `javadoc` before they commit changes to the repo. If there are changes needed, review the differences, and then commit to `master`:

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

- Unit tests are automatically run on commits, so ensure the latest commit's CI build passed.
- FUTURE - Need to run the cross browser platform tests and the performance tests
- Smoke test the UI manually of the component
  - Pay attention to new changes

## Cut Release

If you had to commit changes for the styling, push them to the repo.

Run the following command in the directory of the component's clone. Pass the release version and next development version on the command line and run in batch mode — Maven's interactive prompts fire mid-build and have historically been a source of operator error (ambiguous wording, unpredictable timing). If this is your first time releasing the component, run each `release:` goal separately (`release:clean`, then `release:prepare`, then `release:perform`) so a failure is easier to diagnose and restart.

```sh
$ mvn -B release:clean release:prepare release:perform \
    -DautoVersionSubmodules=true \
    -DreleaseVersion={X.Y.Z} \
    -DdevelopmentVersion={X.Y.Z+1}-SNAPSHOT \
    -Dtag={artifactId}-{X.Y.Z}
```

For example, releasing BookmarksPortlet 3.5.1:

```sh
$ mvn -B release:clean release:prepare release:perform \
    -DautoVersionSubmodules=true \
    -DreleaseVersion=3.5.1 \
    -DdevelopmentVersion=3.5.2-SNAPSHOT \
    -Dtag=BookmarksPortlet-3.5.1
```

The release process creates two commits — one setting the release version (`3.5.1`) and the SCM tag (`BookmarksPortlet-3.5.1`), and one setting the next snapshot (`3.5.2-SNAPSHOT`).

If you need to fall back to the interactive flow (mainly for debugging an unusual `prepare` failure), drop the `-B` and `-D` flags and Maven will prompt for each value during `release:prepare`. Press `Enter` for the defaults.

## Verify and Publish from Central Publisher Portal

Because Maven's `deploy` phase uses the legacy Nexus 2-flavored upload path, the OSSRH Staging API does **not** automatically close the staging repository after `release:perform` uploads the artifacts. You must manually trigger the upload into the Central Portal and then publish from the portal UI.

### Confirm `release:perform` actually deployed

`release:perform` can report success — tagging, committing the next snapshot, building and GPG-signing the artifacts — without its `deploy` step ever reaching Sonatype. The tag and the version bump are then on `master` while nothing is staged, which looks identical to a successful release until you go to publish.

Before promoting, confirm something is actually staged:

```sh
$ curl -4 -s -X GET \
    "https://ossrh-staging-api.central.sonatype.com/manual/search/repositories?ip=any&profile_id={groupId}" \
    -H "Authorization: Bearer $(printf '%s:%s' '{username}' '{password}' | base64 | tr -d '\n')"
```

A repository in `"state":"open"` is what you want. An empty `{"repositories":[]}` means the deploy never landed — re-run **just the deploy** from the tagged checkout rather than re-cutting the release:

```sh
$ cd target/checkout
$ mvn -B deploy -P<the release profiles for this component>
```

This occurred during the `uportal-portlet-parent` 52 release: `release:perform` left a correct signed `.pom` and `.asc` in `target/checkout/target/` and pushed the tag, but the Central Portal reported zero deployments and zero staging repositories. Re-running `deploy` from the checkout uploaded on the first attempt.

One known contributor to a silent `release:perform`: if you keep the GPG passphrase in a `<server><id>gpg.passphrase</id></server>` entry, Maven needs `~/.m2/settings-security.xml` to decrypt it. Without that file Maven logs

```
[WARNING] Failed to decrypt passphrase for server gpg.passphrase:
          ... /.m2/settings-security.xml (No such file or directory)
```

and falls back to `gpg-agent`, which only succeeds while the agent still has the passphrase cached. Releases then pass or fail depending on agent state rather than on anything in the build.

### Push staged artifacts to the portal

Run the following from the **same machine** that ran `release:perform` — the Central Portal matches staging repositories by source IP:

```sh
$ curl -4 -X POST \
    "https://ossrh-staging-api.central.sonatype.com/manual/upload/defaultRepository/{groupId}" \
    -H "Authorization: Bearer $(printf '%s:%s' '{username}' '{password}' | base64 | tr -d '\n')"
```

Replace `{groupId}` with the component's root namespace (e.g. `org.jasig.portlet`, `org.jasig.resourceserver`) and `{username}`/`{password}` with your Central Portal user token credentials (the same ones in `~/.m2/settings.xml`).

**Force IPv4 (`curl -4`).** "Same machine" is necessary but not sufficient. Maven's HTTP client reaches Sonatype over IPv4, so the staging repository is keyed to your IPv4 address. On a dual-stack network `curl` prefers IPv6 and the POST looks up a repository that does not exist:

```
HTTP 400
{"error":"Failed to process request: No repository found for
 <user>/2601:201:…/org.jasig.portlet--default-repository"}
```

while the search above shows the repository sitting `open` under the v4 address (`<user>/203.0.113.7/org.jasig.portlet--default-repository`, your own v4 address). This cost an hour during the `uportal-portlet-parent` 52 release; adding `-4` returned `HTTP 200` immediately and moved the repository to `closed` with a `portal_deployment_id`.

The `| tr -d '\n'` matters for the same reason it does in the release scripts: `base64` wraps at 76 columns on some platforms, and a newline inside the `Authorization` header silently breaks authentication.

If the deployment does not appear in the portal UI, search for open staging repositories:

```sh
$ curl -X GET \
    "https://ossrh-staging-api.central.sonatype.com/manual/search/repositories?ip=any&profile_id={groupId}" \
    -H "Authorization: Bearer $(echo -n '{username}:{password}' | base64)"
```

### Review and publish

1. Log into <https://central.sonatype.com>
2. Navigate to **Deployments** — the staged artifacts should now be visible.
3. Review the release for the expected artifacts (parent POM + every submodule, including any WAR modules).
4. Verify the artifacts pass validation checks (signatures, POM requirements, matching `<packaging>` — see below).
5. Publish the deployment to make it available on Maven Central.

### Recovering from a failed deployment

When the portal's validation rejects a deployment, the staged repository goes into **`failed`** state and **blocks the namespace** until you drop it. New uploads to the same `groupId` (e.g. `org.jasig.portlet`) will return `400 — Repository ... is in state closed/failed and must be dropped before a new release can occur`.

The recovery sequence is the same regardless of which validation rule fired:

1. In the portal UI under **Deployments**, find the failed entry, click **Drop** (not **Publish**).
2. Fix the root cause locally (signature, POM metadata, packaging, etc. — see common failures below).
3. If `release:perform` had already advanced the version (i.e. the `release-plugin` commits + tag are on `master` for a version that never reached Maven Central): the cleanest path is to **skip that version** and release as the next one. Document it in the next-version's release notes ("X.Y.Z was prepared but never published due to ... ; X.Y.Z+1 ships the same intended content"). The orphan tag remains on the repo as historical record. Rewriting tag history is destructive and rarely worth it.
4. Re-run `mvn release:clean release:prepare release:perform`, then the manual upload curl above.

#### Common validation failures

- **`Failed to verify the PGP signature. Please contact support for assistance.`** — The signing key isn't reachable on `keys.openpgp.org` (the keyserver Central Portal queries first). Upload it as described in the Prerequisites and verify with the `curl` check.
- **`Developers information is missing`** — The published POM lacks a `<developers>` element. See "Verify required POM metadata" above.
- **`File with path '.../{artifactId}-{version}.jar' is missing`** — The deployment manifest lists a primary `.jar` for a module that doesn't actually upload one. Common cause for `<packaging>war</packaging>` modules: the Java + War + Spring Boot plugin combo auto-enables a plain `jar` task on top of the War. In Gradle, set `jar.enabled = false` (Spring Boot 1.x) or `bootJar { enabled = false }` (Spring Boot 2+). In Maven this rarely happens because `<packaging>` is authoritative.
- **`Repository ... is in state closed and must be dropped before a new release can occur`** — A previous deployment in the same namespace is still staged. Drop it first.

### A note on POM packaging

Central Portal validation rejects deployments where the advertised `<packaging>` does not match the uploaded artifact's file extension. This bit uPortal during the 2026 release: a Gradle `uploadArchives` block hardcoded `packaging 'jar'` for every submodule, which broke the WAR module. In Maven, `<packaging>` is authoritative per-module, so the bug does not occur the same way — but confirm each module declares the right packaging:

```sh
$ grep -l '<packaging>war</packaging>' $(find . -name pom.xml -not -path '*/target/*')
```

## Create Release Notes

1. Use Git to inspect the incremental commits since the last release (e.g. `$ git log v3.5.0..3.5.1 --no-merges`)
2. Review the issue tracker and confirm that the referenced issues have been Resolved
3. Enter the release notes on the GitHub releases page in the component's repo
   - It's helpful to use the previous release notes as a guide
   - Each commit type goes into a sub section with the type as a header (Fixes, Chores, Features, etc...).

## Update uPortal-start

Open a Pull Request on `uPortal-start` to update the version of the newly released component. Do a quick smoke test to ensure it built:

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
- Be sure to put the release into context for existing deployers to understand.

Have someone with access to the [uPortal Twitter account](https://twitter.com/uportal) announce the release.

## References

- <https://apereo.atlassian.net/wiki/spaces/UPC/pages/102336655/Cutting+a+uPortal+Release>
- <https://central.sonatype.org/publish/publish-portal-guide/>
- <https://central.sonatype.org/publish/publish-portal-ossrh-staging-api/>
- <https://maven.apache.org/maven-release/maven-release-plugin/>
- <https://docs.gradle.org/current/userguide/signing_plugin.html#sec:signatory_credentials>
