# Configuring the Web Security Filter

## Overview

Starting with uPortal 5.14.x, five HTTP Header Security Filters can be configured within the uPortal security property file. In the following documentation, is an overview of the file used to enable the features based on your preferences and its usage.

## Security Headers

The following security headers are available to be enabled/disabled:

* X-Frame-Options (antiClickJacking)
* Content-Security-Policy
* Strict-Transport-Protocol
* X-Content-Type-Option
* Referrer-Policy

## Where to Configure

To customize the security header options, go to the following file in uPortal:

```bash
 ./uPortal-webapp/src/main/resources/properties/security.properties
```

### X-Frame-Options

Also known as antiClickJacking (https://tomcat.apache.org/tomcat-11.0-doc/config/filter.html), clickjacking is an attack that fools users into thinking they are clicking on one thing when they are actually clicking on another.

The following antiClickJacking configurations are included in uPortal, and mimic the configuration options that are also available directly through the tomcat server config file.

* antiClickJackingEnabled
* antiClickJackingOptions
* antiClickJackingUri

Locate the properties below and enable this feature by replacing "false" with "true". The antiClickJackingOptions available are "deny", "sameorigin", or "allow-from". If you select "allow-from", setup the URI for the antiClickJackingUri property. Learn more about which options are best suited for your needs at https://tomcat.apache.org/tomcat-11.0-doc/config/filter.html

```plaintext
# antiClickJackingEnabled:  X-Frame-Options header
sec.anti.click.jacking.enabled=false
# X-Frame-Options: deny, sameorigin, allow-from
sec.anti.click.jacking.options=sameorigin
# If allow-from is selected above, add URI
sec.anti.click.jacking.uri=
```

### Content-Security-Policy

### Strict-Transport-Protocol

### X-Content-Type-Option

### Referrer-Policy
