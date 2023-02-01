## Lifecycle Management

After you register a new portlet, it can be in one of these five states

### Created

### Approved

### Published

Portlet is registered and visible to the intended audience.  You can set an Automatic Expiration Date and Time.  When the time arrives, the portlet will automatically be moved to the Expired state and will not be visible to the audience.

### Expired

### Maintenance

Portlet is installed but not visible to the intended audience.  The portlet is expected to be moved back to Published.

## Changing Portlet State

#### Published to Maintenance

The simple use case is to change the Option from Published to Maintenance and press the Save button.  The scheduled case allows you to schedule when maintenance mode begins and ends.  If Maintenance mode has been scheduled, you will see the scheduled Maintenance Start Date and Time in the Custom Maintenance Message section.  You can click on the Cancelled button to clear out the scheduled maintenance event.

These are the fields for Maintenance Mode

**Custom Message** - displays when a portlet is in maintenance mode

**Stop Immediately** - Checked by default.  If checked, when you press Save, the portlet will immediately enter Maintenance mode.  If not checked, Stop Date and Stop Time must be set

**Stop Date** - disabled unless Stop Immediately is unchecked.  Choose the Date from the dropdown for when Maintenance Mode will start.  If not specified, Save will fail with an error message.

**Stop Time** - disabled unless Stop Immediately is unchecked.  Manually type in the Time, format is hh:mm.  Time is in UTC time.  If not specified, time will default to 00:00 UTC.

**Restart Manually** - Checked by default.  If checked, when you Save the portlet will not restart until the admin changes the option manually back to published.  If not checked, Restart Date and Restart Time must be set

**Restart Date** - disabled unless Restart Manually is unchecked.  Choose the Date from the dropdown for when Maintenance Mode will end, returning the portlet back to Published.  If not specified, Save will fail with an error message.

**Restart Time** - disabled unless Restart Manually is unchecked.  Manually type in the Time, format is hh:mm.  Time is in UTC time.  If not specified, time will default to 00:00 UTC.

#### Maintenance to Published

The simple use case is to change the Option from Maintenance to Published and press the Save button.  If Maintenance mode has a scheduled Restart Date and Restart Time, they will be cleared.