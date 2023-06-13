# Google Analytics 4

Google Universal Analytics support expires on July 1, 2023.  In order to continue with Google
Analytics, new scripts must be deployed to your uPortal environment.  These scripts are included
in the master branch for uPortal and will be available starting release 5.15.0.

If you are not ready to migrate to uPortal 5.15.0, you can copy the needed files into your
uportal-start environment, and they will be included in your deployments as well.

This page will cover the nature of the changes to the scripts and how to manually put the
needed files in your environment, if you are not able to upgrade.

## GA4 Files

Google Analytics 4, like Google Universal Analytics, is implemented in two files within the
uPortal Google Analytics Portlet.

- init.jsp
- up-ga.js

Within the community version of uPortal, these files are located within the uPortal-webapp project

- src/main/webapp/WEB-INF/jsp/GoogleAnalytics/init.jsp
- src/main/webapp/media/skins/common/javascript/uportal/up-ga.js

To use these files without using the uPortal community version but within uportal-start, put these
two files in the overlays folder

- overlays/uPortal/src/main/webapp/WEB-INF/jsp/GoogleAnalytics/init.jsp
- overlays/uPortal/src/main/webapp/media/skins/common/javascript/uportal/up-ga.js

### init.jsp

init.jsp is responsible for inserting the required Google Analytics script into the Google Analytics
portlet that is included in the uPortal render.  It is a bit different from the Google Universal
Analytics implementation in that it requires the Google Analytics Tracking Code to be included
in the script URI.  This requires a bit of code to retrieve the Tracking Code from the
environment.  This file serves the same function in both Google Universal Analytics and Google
Analytics 4, although the implementation is different.

### up-ga.js

up-ga.js is responsible for sending the needed page view, link click,  and timing events to Google Analytics
as the page is rendered and as events within the page are executed.  This file serves the same function
in both Google Universal Analytics and Google Analytics 4, although the implementation is different.

## Comparison

Because of the way that Google Analytics is implemented, there is a significant difference in the calls
that it makes, when compared to Google Universal Analytics.

uPortal using Google Universal Analytics uses up-ga.js to create a tracker with basically default values
(after including the tracker id), and this sends the initial page view.  After that, up-ga.js populates
the tracker's configuration with custom information regarding the type of information, either page view
or timing and then sends the event with a ga("event") call.  All of the data that is included comes from
the tracker's configuration at that time.

uPortal using Google Analytics 4 also uses up-ga.js to create a tracker with basically default values.  However,
this initial event is suppressed.  All events are sent explicitly.  The data needed for the event are included
explicitly within the up.gtag("event") call with the needed data as part of the event body.  The intent is for
the event to be immutable.

The overall structure of the updated up-ga.js script is similar to the original script with several exceptions:

- Several functions return values that will be included in the up.gtag("event") body, rather than put directly
into the configuration of the ga("event")
- Due to dependencies, some functions have been reordered.
- The up.gtag javascript object contains the needed code for executing the needed events, whereas the original script
did not need to be added to the up structure.  At the time, this was required to have visibility into the Google
Analytics 4 functions.  This could be revisited if the current implementation has issues.
- Google Universal Analytics sends individual events.  GA4 collects events and sends them in batch, approximately event
10 seconds.

## Configuration Changes

The biggest change is the need to create and use the Google Analytics 4 tag.  Google Universal Analytics tags begin with
a "UA-" prefix.  GA4 tags begin with a "G-" prefix.  You create the new GA4 tag from within the Google Analytics website within the Admin
function.  Once you have created this new tag, you will need to deploy it to the portlet via a dataImport.

1. Modify the file portlet-definition/google-analytics-config/portlet-definition.xml where you store your portlet definitions.
2. Within that file, replace the UA- value of the "propertyId" field with the new G- prefixed value and save.
3. Perform a dataImport to load the new portlet definition.
