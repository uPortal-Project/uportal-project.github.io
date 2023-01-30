# Onboarding a uPortal Committer

uPortal Committers have three privileges over other contributors.

1. Participate in Committer votes regarding uPortal projects.
2. Review, approve and merge pull requests for uPortal project repos.
3. Cut releases of uPortal projects.

## 1. Committer Voting
Committers should have already joined the uPortal Dev mailing list and been actively
participating. This mailing list is where we currently call for voting.

- uportal-dev@apereo.org

## 2. Managing Pull Requests
Committers help maintain the quality of the uPortal ecosystem by reviewing and approving
pull requests in the various uPortal related repos in GitHub. We have organized them into
three organizations:

1. https://github.com/orgs/uPortal-Project
2. https://github.com/orgs/uPortal-Contrib
3. https://github.com/orgs/uPortal-attic

Each organization should have an existing `uPortal Committers` Team for active
committers. New committers should be added to these teams.

In general, PRs should be reviewed, approved and merged by committers other than the
author. A single committer does not need to perform all of these steps.

## 3. Cutting uPortal-Related Project Releases
The third privilege is cutting releases uPortal, portlet and other projects. We currently
rely on Maven Central to host our official releases, although we do require NPM ???
for web components before they are rolling into a webjar that rests in Maven Central.

### 3.1. Sonatype Set Up
We release through the OSS support provided freely from Sonatype. This requires
that committers obtain an account with Sonatype's service and that an existing committer
request the new committer have rights to make releases.

New committers should create an account:
- https://issues.sonatype.org/secure/Signup!default.jspa
- ... and send their username to the committer granting them access

Existing committers (that are "publishers" in Sonatype) can then request publish
permissions for the new committer:
- https://central.sonatype.org/publish/manage-permissions/
    - Project: Community Support - Open Source Project Repository Hosting (OSSRH)
    - Type: Publishing Support
    - Summary: Grant <committer's login> access to org.jasig
    - Description: Please grant user <committer's login> access to push releases to groupId org.jasig
    - Group Id: org.jasig
    - Published Hostname: oss.sonatype.org
    - Username(s): <committer's login>
    - Modify publishing permissions?: yes

### 3.2 NPM Set Up
We also need to release our web components to NPM before they can be rolled into
webjars.

New committers should create an account:
- https://www.npmjs.com/signup
- ... and send their username to the committer granting them access

Existing committer with enough privileges can add the new committer:
- https://www.npmjs.com/settings/uportal/teams/team/developers/users

