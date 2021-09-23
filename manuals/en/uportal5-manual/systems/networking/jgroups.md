# JGroups -- Ehcache Sharing Across the Cluster

## Purpose

JGroups is used to share caching judiciously across the cluster.
The most important caches are the CAS ticket ones that require
all servers to be able to handle CAS service checks even from
a server that the initiating user is NOT using.

Sharing other caches is mostly focused on a consistent service,
such as the update of a layout.

## Disabling JGroups (uPortal 5.11.1+)

Some implementers have found they do not need support for CAS cache
replication, and have opted to forego JGroups. JGroups adds
to start up time and network configuration/traffic.

Another reason to disable JGroups is in dev/demo setups where
there is only one server. In such cases JGroups adds nothing.

The easiest way to disable JGroups is to use the Ehcache file
that omits all the JGroups setup. This can be done by setting
the Ehcache configuration file property to `ehcache-no-jgroups.xml`
in uPortal.properties for the cluster.

```Properties
org.apereo.portal.ehcache.filename=ehcache-no-jgroups.xml
```

If an implementer has modified their `ehcache.xml`, we suggest
creating a duplicate file with the name above (i.e. `ehcache-no-jgroups.xml`)
and removing the JGroup listeners and JGroups Provider Factory.
