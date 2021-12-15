# log4j-elasticbeanstalk-remove
Scripts to remove unused log4j.jar from AWS Elastic Beanstalk Tomcat images and mitigate CVE-2021-44228 Log4Shell attacks.

## The issue
CVE-2021-44228 is a serious security issue for users of Log4j. This applies to any Java application that is actively using log4j-impl.jar .

Many Java applications contain or use log4j-api.jar, which only contain the API definitions, not the actual code implementation, so having only log4j-api.jar is not a security issue, despite some (even official) tools reporting that as an issue. 

## Impact on Tomcat and Webapps

Tomcat is a container, it is a Java application itself, but it then loads and executes webapps, which are also Java applications. 

If log4j implementation is used in either Tomcat or a webapp inside that Tomcat, then the server may be exposed to the CVE-2021-44228 security issue.

Moreover, is log4j is available inside Tomcat "common" or "shared" library folder (currently a folder named `lib` in the Tomcat home), then that version in the common library folder will be used instead of the version provided by the webapps. This is specifically dangerous, cause you may update log4j version in your webapp, but miss the fact the actually used log4j is an older version loaded from tomcat common library folder.

## AWS Elastic Beanstalk images

By default, Tomcat does not use and does not ship log4j (see here for details http://mail-archives.apache.org/mod_mbox/www-announce/202112.mbox/%3C028d1058-2e48-f72c-2037-2070d73b7411@apache.org%3E )

However, AWS Tomcat 8.5 images seems to ship log4j.jar in `/usr/share/java/log4j.jar`, which is then linked inside `/usr/share/tomcat/lib/log4j.jar`.

Notice that `/usr/share/tomcat/lib/` is the common library folder, so this version of log4j may end up being used instead of the (potentially updated and more secure) version of log4j available in your webapps. 

Tomcat in Elastic Beanstalk images is not configured to use log4j for logging, so there is no specific reason for that jar to be there. While not used, the jar is loaded by the JVM. 

This by itself should not pose a significant security risk, until log4j is activated somehow, or a webapp using a more recent version does not end up using the non-updated version.

## What is inside this repository

An Elastic Beanstalk configuration that removes the shared log4j.jar file. 

See https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/customize-containers-ec2.html for documentation and how to use these files. 

It worked for me so I'm sharing it, but make sure to test your webapps after applying this script. 

## How can I help

Please report if it worked for you, open but if it didn't. 

Please contibute patches if you adapted it for other Elastic Beanstalk images or other versions of Tomcat or Java.
