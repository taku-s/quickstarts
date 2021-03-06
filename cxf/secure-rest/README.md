secure-rest: Demonstrates a Secure REST Web service
===============================================
Author: Fuse Team  
Level: Beginner  
Technologies: Fuse, OSGi, CXF  
Summary: This quickstart demonstrates how to create a secure RESTful (JAX-RS) web service using CXF and expose it through the OSGi HTTP Service.
Target Product: Fuse  
Source: <https://github.com/jboss-fuse/quickstarts>

What is it?
-----------
This quick start demonstrates how to create a secure RESTful (JAX-RS) web service using CXF and expose it with the OSGi HTTP Service.

In studying this quick start you will learn:

* how to configure the JAX-RS web services by using the blueprint configuration file.
* how to secure the web service by using the blueprint configuration file.
* how to use JAX-RS annotations to map methods and classes to URIs
* how to use JAXB annotations to define beans and output XML responses
* how to use the JAX-RS API to create HTTP responses

For more information see:

* https://access.redhat.com/site/documentation/JBoss_Fuse/ for more information about using JBoss Fuse

System requirements
-------------------
Before building and running this quick start you need:

* Maven 3.1.1 or higher
* JDK 1.7 or 1.8
* JBoss Fuse 6

### How and What is secured

In this example we are using a CXF interceptor that ensures that a request has been authenticated before allowing it to pass. For
performing the authentication, this interceptor will delegate to JAAS, using the realm name 'karaf'. In this way we will 
reuse the same authentication mechanism that is being used to secure other Fuse facilities, such as the remote
SSH shell and the webconsole. You can see the definition [here](https://github.com/jboss-fuse/fabric8/blob/1.2.0.redhat-6-3-x/quickstarts/cxf/secure-rest/src/main/resources/OSGI-INF/blueprint/blueprint.xml#L71-L74). In the [test](https://github.com/jboss-fuse/fabric8/blob/1.2.0.redhat-6-3-x/quickstarts/cxf/secure-rest/src/test/java/io/fabric8/quickstarts/rest/secure/CrmSecureTest.java#L72-L80) you'll see that we are using a Basic Authentication Scheme: this is not a good practice, so don't try this in your production enviroment. The example just shows how you can secure your application the simpler way.

Build and Deploy the Quickstart
-------------------------------

1. Change your working directory to `secure-rest` directory.
* Run `mvn clean install` to build the quickstart.
* Verify `etc/users.properties` from the JBoss Fuse installation contains the following 'admin' user configured: `admin=admin,admin` (it is commented by default)
* Start JBoss Fuse 6 by running bin/fuse (on Linux) or bin\fuse.bat (on Windows).
* In the JBoss Fuse console, enter the following command:

        osgi:install -s mvn:org.jboss.quickstarts.fuse/cxf-secure-rest/6.3.0.redhat-187

* Fuse should give you an id when the bundle is deployed
* You can check that everything is ok by issuing  the command:

        osgi:list
   your bundle should be present at the end of the list

Use the bundle
--------------

### Browsing Web service metadata

A full listing of all CXF web services is available at

    http://localhost:8181/cxf

After you deployed this quick start, you will see the following endpoint address appear in the 'Available RESTful services' section:

    http://localhost:8181/cxf/securecrm/
**Note:**: Don't try to access this endpoint address from browser, as it's inaccessible by design

Just below it, you'll find a link to the WADL describing all the root resources:

    http://localhost:8181/cxf/securecrm/?_wadl

You can also look at the more specific WADL, the only that only lists information about 'customerservice' itself:

	http://localhost:8181/cxf/securecrm/customerservice?_wadl&_type=xml

### Access services using a web browser

You can use any browser to perform a HTTP GET.  This allows you to very easily test a few of the RESTful services we defined:

Use this URL to display the XML representation for customer 123:

    http://localhost:8181/cxf/securecrm/customerservice/customers/123

Because we need to pass along credentials to actually access the service in this security-enabled quick start, the browser will popup a dialog to let you input user/password(admin/admin by default)

**Note:** if you use Safari, you will only see the text elements but not the XML tags - you can view the entire document with 'View Source'

### To run the tests:

In this quick start project, we also provide integration tests which perform a few HTTP requests to test our Web services. We
created a Maven `test` profile to allow us to run tests code with a simple Maven command after having deployed the bundle to Fuse:

1. Change to the `secure-rest` directory.
2. Run the following command:

        mvn -Ptest

The tests in `src/test/java/org.jboss.quickstarts.fuse.rest.secure/CrmSecureTest` make a sequence of authenticated RESTful invocations and displays the results.

### To run a command-line utility:

You can use a command-line utility, such as cURL or wget, to perform the HTTP requests.  We have provided a few files with sample XML representations in `src/test/resources`, so we will use those for testing our services.

1. Open a command prompt and change directory to `secure-rest`.
2. Run the following curl commands (curl commands may not be available on all platforms):

    * Create a customer

            curl --basic -u admin:admin -X POST -T src/test/resources/add_customer.xml -H "Content-Type: text/xml" http://localhost:8181/cxf/securecrm/customerservice/customers

    * Retrieve the customer instance with id 123

            curl --basic -u admin:admin http://localhost:8181/cxf/securecrm/customerservice/customers/123

    * Update the customer instance with id 123

            curl --basic -u admin:admin -X PUT -T src/test/resources/update_customer.xml -H "Content-Type: text/xml" http://localhost:8181/cxf/securecrm/customerservice/customers

    * Delete the customer instance with id 123

             curl --basic -u admin:admin -X DELETE http://localhost:8181/cxf/securecrm/customerservice/customers/123

### Managing the user credentials

You can define additional users in the JAAS realm in two ways:

1. By editing the `etc/users.properties` file, adding a line for every user your want to add (syntax: `user = password, roles`)

            myuser = mysecretpassword

2. Using the `jaas:` commands in the JBoss Fuse console:

            jaas:manage --realm karaf --index 1
            jaas:useradd myuser mysecretpassword
            jaas:update

### Changing /cxf servlet alias

By default CXF Servlet is assigned a '/cxf' alias. You can change it in a couple of ways

1. Add `org.apache.cxf.osgi.cfg` to the `/etc` directory and set the `org.apache.cxf.servlet.context` property, for example:

        org.apache.cxf.servlet.context=/custom

2. Use shell config commands, for example:

        config:edit org.apache.cxf.osgi
        config:propset org.apache.cxf.servlet.context /custom
        config:update

Undeploy the Bundle
-------------------

To stop and undeploy the bundle in Fuse:

1. Enter `osgi:list` command to retrieve your bundle id
2. To stop and uninstall the bundle enter

        osgi:uninstall <id>

