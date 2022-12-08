# webMethods
This is a work in progress.  The default setup includes Logs and Errors Inbox to help you troubleshoot the webMethods Integration server.

## Prerequisites
1. Download complete java agent
2. Software AG webMethods Integration Server installed

## Setup
1. Download and extract the New Relic java agent somewhere.  In this case, `newrelic-java.zip` was extracted to `C:\newrelic`
2. Edit C:\SoftwareAG\profiles\IS_default\configuration\wrapper.conf and add a line `wrapper.java.additional.10=-javaagent:C:\newrelic\newrelic.jar`
3. Restart webMethods Integration Server
4. Check http://localhost:5555/WmAdmin/ - the default username is `Administrator` and password is `manage`
   
## Configuration
1. Create a new folder called `extensions` in `C:\newrelic`
2. Create a new file `webMethods.xml` and put it inside `C:\newrelic\extensions`

Your New Relic Java agent directory structure should look like this:
```
C:.
|   extension-example.xml
|   extension.xsd
|   LICENSE
|   newrelic-api-javadoc.jar
|   newrelic-api-sources.jar
|   newrelic-api.jar
|   newrelic.jar
|   newrelic.yml
|   THIRD_PARTY_NOTICES.md
|
+---extensions
|       webMethods.xml
|
\---logs
        newrelic_agent.log
```

### webMethods.xml
This file will be responsible for getting transactions from webMethods.  This is a work-in-progress, and will need to be fine-tuned.

```
<?xml version="1.0" encoding="UTF-8"?>

<extension xmlns="https://newrelic.com/docs/java/xsd/v1.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="newrelic-extension extension.xsd" name="webMethods-10.11"
  version="1.0" enabled="true">

  <instrumentation metricPrefix="webMethods_">

    <pointcut transactionStartPoint="true" ignoreTransaction="false" excludeFromTransactionTrace="false">
      <nameTransaction/>
      <className includeSubclasses="true">com.wm.app.b2b.server.JavaService</className>
      <method>
        <name>baseInvoke</name>
      </method>
    </pointcut>

    <pointcut transactionStartPoint="true" ignoreTransaction="false" excludeFromTransactionTrace="false">
      <nameTransaction/>
      <className includeSubclasses="true">com.wm.app.b2b.server.FlowSvcImpl</className>
      <method>
        <name>baseInvoke</name>
      </method>
    </pointcut>

    <pointcut transactionStartPoint="true" ignoreTransaction="false" excludeFromTransactionTrace="false">
      <nameTransaction/>
      <className includeSubclasses="true">com.wm.lang.flow.FlowInvoke</className>
      <method>
        <name>invoke</name>
      </method>
    </pointcut>

    <pointcut transactionStartPoint="true" ignoreTransaction="false" excludeFromTransactionTrace="false">
      <nameTransaction/>
      <className includeSubclasses="true">com.wm.lang.flow.FlowState</className>
      <method>
        <name>invoke</name>
      </method>
    </pointcut>

  </instrumentation>
</extension>
```


## Dashboard