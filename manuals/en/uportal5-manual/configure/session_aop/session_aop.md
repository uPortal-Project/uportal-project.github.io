# Session Replication AOP

In order for Apache Tomcat to successfully perform session replication, it is necessary
that all classes stored in the session are serializable.  If the class is not serializable,
the class cannot be successfully stored in the session and reconstituted on the other
tomcat servers.

However, uPortal does not verify if a class being stored in a session is in fact serializable
before writing it.  Therefore, uPortal has an additional feature that will allow the
uPortal administrator to configure logging for classes that are not serializable and to address
those classes.  This feature is Optional.  If it is not enabled, uPortal will continue to behave
as normal, and the user can continue with Sticky Sessions to ensure that session information is
available for the user.  However, if this feature is enabled, additional logging becomes available.

# Enabling Logging

To enable logging, in uPortal-start, go to the gradle.properties file and set "deploySessionReplicationAop"
to true.  Then, the next time that the gradlew task of tomcatConfig is run, logging will be enabled.
The tomcatConfig task is run whenever any of these tasks are run: portalInit, tomcatInstall, or tomcatConfig.

# Viewing the logs

All logs are by default created in the $CATALINA_HOME/logs folder.  The log in question is aop-serializer.log.
At the end of the log, you will see a line like "value <class name> is NOT serializable" for every time a class
that is not serializable is written to the session.

# Under the covers

The process that occurs as part of tomcatConfig is as follows:

1. if "deploySessionReplicationAop" is true, run the gradlew task deployAopArchives.  This task is contained in
the file gradle/tasks/aop.gradle
1. deployAopArchives runs several tasks:
	- download dependencies - we need an AspectJ jar to be available for tomcat
	- build the AOP jar - we need to compile a java AspectJ class HttpSessionAspect, and we need to
	weave AspectJ commands into the Tomcat class that implements HttpSession.
	- build the Catalina AOP Jar - we need to remove the class that implements HttpSession from 
	the catalina jar, since we can't have two copies on the classpath.
	- Put the needed files in the correct place, including the newly built jars and the logging configuration.
	
# AOP

We use AspectJ to add capabilities to uPortal without having to modify any uPortal code.  There are two classes
that are required to be modified, and the deployAopArchives task is responsible for making any modifications.
This document will not cover in any details how AspectJ works, but it will cover the changes that are being made.

## HttpSessionAspect

HttpSessionAspect.java is the joinpoint describing what classes need to be modified and what to happen once the
modified class is executed.  This class is responsible for writing to the log once the AspectJ class is called.
If you look at the code, you can see on line 42

```
    @Before("execution(* javax.servlet.http.HttpSession+.setAttribute(..))")
```

This block says the HttpSessionAspect class will be called before a class that implements HttpSession and executes
the setAttribute method.

The rest of the code is for logging any needed information.

## HttpSession

Within Apache Tomcat, the class that implements HttpSession is org.apache.catalina.sessions.StandardSessionFacade.
The deployAopArchives gradle task is aware of which class will match the above JoinPoint and as part of the compileJava
task will run the AspectJ post-compile weaving job, controlled by the "io.freefair.aspectj.post-compile-weaving" plugin,
which is found in the custom/aop/build.gradle project file.  When the compileJava task is executed, it will compile
both the HttpSessionAspect class AND integrate the AspectJ joinPoint into the StandardSessionFacade class.

# AspectJ v Spring AOP

During development, Spring AOP was tried unsuccessfully.  The reason Spring AOP failed was because it can only manage
Spring Beans (i.e. anything with the @Bean or other annotation, or defined in Spring configuration files).
StandardSessionFacade is not managed by Spring; it is more fundamental, living within Tomcat itself.  Therefore,
AspectJ had to be used.  You'll also notice that the scripts write files directly into the $CATALINA_HOME/lib folder
for jars that are created or modified, instead of within $CATALINA_HOME/webapps/uPortal, for the same reason.
