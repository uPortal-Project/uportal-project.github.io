# Health Check - http://localhost:8080/uPortal/health-check

## Purpose

Provide a URL that will respond with `200` code when uPortal is running fine and `500` otherwise.

## Basic Check -- http://localhost:8080/uPortal/health-check

This replies with `200` when a basic controller can resolve correctly. The response content will be an empty JSON hashmap.

## Additional Checks

The Health Check feature (as of uPortal 5.11.0+) now supports additional checks. There are two built in with
a simple interface to implement additional checks as desired.

Multiple checkers can be invoked by adding their identifier in a common-separated list to `detail` URL parameter.
For example:

```
   http://localhost:8080/uPortal/health-check?detail=MEMORY,DB
```

### http://localhost:8080/uPortal/health-check?detail=ALL

Adding `?detail=ALL` will trigger a call to all beans that implement `IHealthChecker` in the uPortal Spring context.

### http://localhost:8080/uPortal/health-check?detail=MEMORY

The `MEMORY` Health Check will add several memory values to the output, such as free, used and total memory
for the current node's JVM. Note that these values will vary between nodes in the cluster.

### http://localhost:8080/uPortal/health-check?detail=DB

The `DB` Health Check will trigger a call of the Hibernate SQL test to confirm that a connection can be made
and SQL can be run. This will timeout after 5 seconds.

## New Checks

New checks can be implemented by extending the `IHealthChecker` Interface and adding a Spring bean of the class implementation.

Please consider donating new general checkers back to the community.

