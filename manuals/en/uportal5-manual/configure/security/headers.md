# Configuring the Web Security Filter

## Overview

Starting with uPortal 5.14.x, five HTTP Header Security Filters can be configured within the uPortal security property file. In the following documentation, is an overview of the file used to enable the features based on your preferences and its usage.

## Security Headers

The following security headers that are available to be enabled/disabled include:

* X-Frame-Options (antiClickJacking)
* Content-Security-Policy
* Strict-Transport-Protocol
* X-Content-Type-Option
* Referrer-Policy

## Where to Configure

To customize the security header response options, go to the following file in uPortal:

```bash
 ./uPortal-webapp/src/main/resources/properties/security.properties
```

### X-Frame-Options

Also known as antiClickJacking (https://tomcat.apache.org/tomcat-11.0-doc/config/filter.html), clickjacking is an attack that fools users into thinking they are clicking on one thing when they are actually clicking on another.

The following antiClickJacking configurations are included in uPortal, and mimic the configuration options that are also available directly through the tomcat server config file.

* antiClickJackingEnabled
* antiClickJackingOptions
* antiClickJackingUri

Locate the code block below and enable this feature by replacing "false" with "true". The antiClickJackingOptions available are "deny", "sameorigin", or "allow-from". If you select "allow-from", setup the URI for the antiClickJackingUri property. Learn more about which options are best suited for your needs at https://tomcat.apache.org/tomcat-11.0-doc/config/filter.html

```plaintext
 # antiClickJackingEnabled:  X-Frame-Options header
 sec.anti.click.jacking.enabled=false
 # X-Frame-Options: deny, sameorigin, allow-from
 sec.anti.click.jacking.options=sameorigin
 # If allow-from is selected above, add URI
 sec.anti.click.jacking.uri=
```

### Content-Security-Policy

The Content-Security-Policy header allows you to restrict which resources can be loaded and the URLs that can they can be loaded from.

By default, the content security policy is disabled in uPortal, but to enable it go to the security.properties file and switch the sec,content.sec.policy.enabled property to "true". To define the policy, see the documentation at https://content-security-policy.com/ to find the policy that works best for your purpose.

```plaintext
 # Content-Security-Policy: default-src, script-src, style-src, img-src
 # See more details at: https://content-security-policy.com/
 sec.content.sec.policy.enabled=false
 sec.content.sec.policy=default-src 'self'
```

### Strict-Transport-Protocol

Often referred to as HSTS, this response header informs the browser the site should only be accessed uding HTTPS,
and that any attempts to access it using HTTP should be automatically be converted to HTTPS.

There are a few directives that can be customized. You can refer to the documentation at https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security for guidance. By default, the protocol is disabled and can be configured in the
security.properties file. The following code block is the section you will customize in the properties file.

```plaintext
 # Strict-Transport-Security: max-age=###; includeSubDomains; preload
 sec.hsts.enabled=false
 sec.hsts.maxage.seconds=31536000
 sec.hsts.include.subdomains=true
 sec.hsts.preload=false
```

### X-Content-Type-Options

The X-Content-Type-Options response header is a marker used by the server to indicate that the MIME types advertised in the
Content-Type headers should be followed and not be changed.

In the security.properties file, set the sec.x.content.type.enabled property to "true" to enable the response header. The only directive used in this property is "nosniff", which will be used if the property is enabled. Further documentation on this security response header can be found at: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Content-Type-Options

```plaintext
 # X-Content-Type-Options: "nosniff" will be used if enabled is set to true
 sec.x.content.type.enabled=false
```

### Referrer-Policy

Referrer-Policy controls how much referrer information should be included with requests.

In the following code block located in the security.properties file, you can enable the header response by setting
sec.referrer.policy.enabled to "true". In the sec.referrer.policy you may define your policy based on your preference by
following the syntax defined at https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy

```plaintext
 # Referrer-Policy available directives to pass include:
 # See more details at: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy
 sec.referrer.policy.enabled=false
 sec.referrer.policy=no-referrer
```
