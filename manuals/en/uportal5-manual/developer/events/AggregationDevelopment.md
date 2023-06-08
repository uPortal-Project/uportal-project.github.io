# Aggregation for Raw Events



## Writing a new Aggregator

`PortalRawEventsAggregatorImpl` is the driver behind the aggregation process for Raw Events.  The method `doAggregateRawEvents` handles transaction management and calls `doAggregationRawEventsInternal`. `doAggregationRawEventsInternal` is the method that iterates over all of the newly processed events.

Events are selected within the `portalEventDao.aggregatePortalEvents()` call, which passes in a `AggregateEventsHandler` lambda to be called for each event. `portalEventDao` refers to an interface `IPortalEventDao`, which is implemented as a bean by `JpaPortalEventStore`.  This class finds all of the events to process and iterates over each event,
passing the AggregateEventsHandler as a parameter.

AggregateEventsHandler is defined within PortalRawEventsAggregateImpl as an internal class.  Its job is to identify all of the aggregates that are defined and call each of them for every record.  In the default uPortal-start environment, there are several aggregators that are called for each event.

AggregateEventsHandler is a lambda, and its magic occurs in the apply() method.  It has two interesting phases.

1. For IntervalAwarePortalEventAggregator classes, it will make sure that handle time boundary crossings are cleaned up.  For example, if a record is processed at 1:59:59 and the next record is processed at 2:00:01, this would cross the HOUR boundary and force the previously aggregated records that recognize that boundary crossing to be flushed.
2. Every class that implements IntervalAwarePortalEventAggregator or SimplePortalEventAggregator is processed for each record, based on which classes are available as beans to the application.

### SimplePortalEventAggregator

This is a simple aggregator, as it does not care about time interval boundary crossings by default. It simply executes the class's aggregateEvent method over the event. This logic can be as simple or as complicated as defined. The simplest implementation is the `org.apereo.portal.events.aggr.LoggingPortalEventAggregator`, which simply writes to the log when an event is processed. There is no additional filtering or logic.

A slightly more complicated simple aggregator is `org.apereo.portal.events.aggr.analytics.AnalyticsEventAggregator`. This aggregator simply grabs the data structure for the event and writes it to its own table `UP_ANALYTICS_EVENTS`, managed by class `org.apereo.portal.events.handlers.db.PersistentAnalyticsEvent`.

### IntervalAwarePortalEventAggregator

Classes that implement this interface are the main reason for the aggregator.  They are designed to group events into a single record, which is then persistent into another UP Aggregation table.  Much configuration is provided for free by the PortalRawEventsAggregatorImpl class.  Things like transaction management and time boundaries are automatically handled.  One significant example that implements this interface is org.apereo.portal.events.aggr.portletexec.PortletExecutionAggregator.

Refer to https://apereo.atlassian.net/wiki/spaces/UPM41/pages/103942255/Event+Logging for additional information.


