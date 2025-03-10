[//]: # (title: Service Messages)
[//]: # (auxiliary-id: Service Messages)

_Service messages_ are specially constructed pieces of text that pass commands/information about the build from the build script to the TeamCity server.

To be processed by TeamCity, __they need to be written to the standard output stream of the build, that is printed or echoed from a build step__.

>It is recommended to output one message per line (by dividing them with `\n` symbols) and flush the stdout buffer after each service message.

Examples:

<tabs>

<tab title="Windows">

```Shell
echo ##teamcity[<messageName> 'value']

``` 
</tab>

<tab title="Linux">

```Shell
echo "##teamcity[<messageName> 'value']"

``` 
</tab>

<tab title="PowerShell script">

```Shell
Write-Host "##teamcity[<messageName> 'value']"

``` 
</tab>

</tabs>

A single service message cannot contain a newline character inside it, it cannot span across multiple lines.

## Service Messages Formats

Service messages support two formats:

* Single-attribute message:
    ```Shell
    ##teamcity[<messageName> 'value']
    
    ```

* Multiple-attribute message:
    ```Shell
    ##teamcity[<messageName> name1='value1' name2='value2']
    
    ```

  Multiple-attribute message can more formally be described as:   
  ```Shell
    ##teamcity[messageNameWSPpropertyNameOWSP=OWSP'value'WSPpropertyName_IDOWSP=OWSP'value'...OWSP]
    
    ```

where:
 * `messageName` is the name of the message, such as `flowStarted` or `setParameter`.
 * `propertyName` is a name of the message attribute. Must be a valid Java ID.
 * `value` is a value of the attribute. Must be an [escaped value](#Escaped+Values).
 * `WSP` is a required whitespace(s): space or tab character (`\t`).
 * `OWSP` is an optional whitespace(s).
 * `...` is any number of `WSPpropertyNameOWSP=OWSP'value'` blocks.
 
## Escaped Values

For escaped values, TeamCity uses a vertical bar `|` as an escape character. To have certain special characters properly interpreted by the TeamCity server, they must be preceded by a vertical bar. For example, the following message:

```Shell
##teamcity[testStarted name='foo|'s test']

```

will be displayed in TeamCity as `foo's test`. Refer to the table of the escaped values below.

<table>

<tr>

<td>

Character

</td>

<td>

Escape as

</td>

</tr>

<tr>

<td>

' (apostrophe)

</td>

<td>

|'

</td></tr>

<tr>

<td>

\n (line feed)

</td>

<td>

|n

</td></tr>

<tr>

<td>

\r (carriage return)

</td>

<td>

|r

</td></tr>

<tr>

<td>

\uNNNN (unicode symbol with code 0xNNNN)

</td>

<td>

|0xNNNN

</td></tr>

<tr>

<td>

| (vertical bar)

</td>

<td>

||

</td></tr>

<tr>

<td>

[ (opening bracket)

</td>

<td>

|[

[//]: # (Internal note. Do not delete. "Build Script Interaction with TeamCityd44e192.txt")    

</td></tr>

<tr>

<td>

] (closing bracket)

</td>

<td>

|]

</td></tr></table>

### Message FlowId

Any message supports the optional attribute `flowId`. In the following examples, `<messageName>` is the name of the specific service message.

`flowId` is a unique identifier of the message flow in a build. Flow tracking is necessary, for example, to distinguish separate processes running in parallel. The identifier is a string that must be unique in the scope of an individual build.

```Shell
##teamcity[<messageName> flowId='flowId' ...]

```

In most cases, `flowId` is the only attribute required to start a message flow.

When you absolutely need to start a flow not inside the root flow but as a subflow inside an existing flow, add the `flowStarted` parameter and specify the parent flow ID as the `parent` parameter. Flows without the specified parent start inside the root flow of the current step.   
To end a subflow, use the `flowFinished` parameter. Ending a parent flow automatically closes all its subflows, but we recommend declaring the flow order explicitly:

```Shell
##teamcity[flowStarted flowId='MainFlow' ...]
##teamcity[flowStarted flowId='SubFlow1' parent='MainFlow' ...]
##teamcity[flowFinished flowId='SubFlow1' ...]
##teamcity[flowStarted flowId='SubFlow2' parent='MainFlow' ...]
##teamcity[flowFinished flowId='SubFlow2' ...]
##teamcity[flowFinished flowId='MainFlow' ...]

```

This custom subflow order affects the sequence of reports in a build log and allows reporting results as subtrees.

## Reporting Messages to Build Log

You can report messages to a build log as follows:

```Shell
##teamcity[message text='<message text>' errorDetails='<error details>' status='<status value>']

```

where:
* `status` can take the following values: `NORMAL` (default), `WARNING`, `FAILURE`, `ERROR`.
* `errorDetails` is used only if `status` is `ERROR`, in other cases it is ignored.   
This message fails the build in case its status is `ERROR` and the "_Fail build if an error message is logged by build runner_" box is checked on the __[Build Failure Conditions](build-failure-conditions.md)__ page of the build configuration. For example:
    ```Shell
    ##teamcity[message text='Exception text' errorDetails='stack trace' status='ERROR']
    
    ```
  
>If you want to control when a build fails, you can either configure a [failure condition](build-failure-conditions.md) or send a special [message with a build problem](#Reporting+Build+Problems).

### Blocks of Service Messages

Blocks are used to group several messages in the build log.

Block opening:

The `blockOpened` system message has the `name` attribute, and you can also add its description: 

```Shell
 ##teamcity[blockOpened name='<blockName>' description='<this is the description of blockName>']

```

Block closing:

```Shell
 ##teamcity[blockClosed name='<blockName>']

```

>Note that when you close a block, all its inner blocks are closed automatically.
>
{type="note"}

### Reporting Compilation Messages

```Shell
##teamcity[compilationStarted compiler='<compiler_name>']
...
##teamcity[message text='compiler output']
##teamcity[message text='compiler output']
##teamcity[message text='compiler error' status='ERROR']
...
##teamcity[compilationFinished compiler='<compiler name>']

```

where:
* `compiler_name` is an arbitrary name of the compiler performing compilation, for example, `javac` or `groovyc`. Currently, it is used as a block name in the build log.
* Any message with status `ERROR` reported between `compilationStarted` and `compilationFinished` will be treated as a compilation error.

### Reporting Tests

To use the TeamCity on-the-fly test reporting, a testing framework needs dedicated support for this feature to work (alternatively, [XML Report Processing](xml-report-processing.md) can be used). If TeamCity doesn't support your testing framework natively, it is possible to modify your build script to report test runs to the TeamCity server using service messages. This makes it possible to display test results in real-time, make test information available on the __[Tests](build-results-page.md#Tests+Tab)__ tab of the __Build Results__ page.

### Message Creation Timestamp

Test report messages support the optional attribute `timestamp`. In the following examples, `<messageName>` is the name of the specific service message.

```Shell
##teamcity[<messageName> timestamp='timestamp' ...]

```

The timestamp format is `yyyy-MM-dd'T'HH:mm:ss.SSSZ` or `yyyy-MM-dd'T'HH:mm:ss.SSS` according to [Java SimpleDateFormat syntax](http://docs.oracle.com/javase/1.5.0/docs/api/java/text/SimpleDateFormat.html).

Example:

```Shell
##teamcity[<messageName> timestamp='2008-09-03T14:02:34.487+0400' ...]
##teamcity[<messageName> timestamp='2008-09-03T14:02:34.487' ...]

```

For .NET [DateTimeOffset](https://msdn.microsoft.com/en-us/library/az4se3k1(v=vs.110).aspx), a code like this
```JavaScript
var date = DateTimeOffset.Now;
var timestamp = $"{date:yyyy-MM-dd'T'HH:mm:ss.fff}{date.Offset.Ticks:+;-;}{date.Offset:hhmm}";

```

will result in
```Shell
##teamcity[<messageName> timestamp='2008-09-03T14:02:34.487+0400' ...]

```

#### Supported Test ServiceMessages

__Test suite messages:__ Test suites are used to group tests. TeamCity displays tests grouped by suites on the __[Tests](build-results-page.md#Tests+Tab)__ tab of the __Build Results__ page and in other places.

```Shell
##teamcity[testSuiteStarted name='suiteName']
<individual test messages go here>
##teamcity[testSuiteFinished name='suiteName']

```

All the individual test messages are to appear between `testSuiteStarted` and `testSuiteFinished` (in that order) with the same `name` attributes.

#### Nested Test Reporting

Starting another test finishes the currently started test in the same _flow_. To still report tests from within other tests, you will need to specify another [`flowId`](#Message+FlowId) in the nested test service messages.

__Test start/stop messages:__

```Shell
##teamcity[testStarted name='testName' captureStandardOutput='<true/false>']
<here go all the test service messages with the same name>
##teamcity[testFinished name='testName' duration='<test_duration_in_milliseconds>']

```

Indicates that `testName` was run. If the `testFailed` message is not present, the test is regarded successful, where
* `duration` (optional numeric integer attribute) sets the test duration in milliseconds to be reported in TeamCity UI. If omitted, the test duration will be calculated from the messages timestamps. If the timestamps are missing — from the actual time the messages were received on the server.
* `captureStandardOutput` (optional boolean attribute) — if `true`, all the standard output and standard error messages received between `testStarted` and `testFinished` messages will be considered test output. The default value is `false`: assumes usage of `testStdOut` and `testStdErr` service messages to report the test output.

<note>

All the other test messages (except for `testIgnored`) with the same `name` attribute must appear between the `testStarted` and `testFinished` messages (in that order).   

If using Ant's `echo` task to output the messages, make sure to include the `flowId` attribute with the same value in all the messages related to the same test / test suite as otherwise they [will not be processed correctly](http://jetbrains.net/tracker/issue/TW-5059).
</note>

It is highly recommended that you ensure that the pair of `test suite` + `test name` is unique within the build. For advanced TeamCity test-related features to work, test names must not deviate from one build to another (a single test must be reported under the same name in every build). Include absolute paths in the reported test names is __strongly discouraged__.

__Ignored tests:__

```Shell
##teamcity[testIgnored name='testName' message='ignore comment']

```

Indicates that `testName` is present but was not run (was ignored) by the testing framework. As an exception, the `testIgnored` message can be reported without the matching `testStarted` and `testFinished` messages.

__Test output:__

```Shell
##teamcity[testStarted name='className.testName']
##teamcity[testStdOut name='className.testName' out='text']
##teamcity[testStdErr name='className.testName' out='error text']
##teamcity[testFinished name='className.testName' duration='50']

```

The `testStdOut` and `testStdErr` service messages report the test's standard and error output to be displayed in the TeamCity UI. There must be only one `testStdOut` and one `testStdErr` message per test. An alternative but a less reliable approach is to use the `captureStandardOutput` attribute of the `testStarted` message.

__Test result:__


```Shell
##teamcity[testStarted name='MyTest.test1']
##teamcity[testFailed name='MyTest.test1' message='failure message' details='message and stack trace']
##teamcity[testFinished name='MyTest.test1']

##teamcity[testStarted name='MyTest.test2']
##teamcity[testFailed type='comparisonFailure' name='MyTest.test2' message='failure message' details='message and stack trace' expected='expected value' actual='actual value']
##teamcity[testFinished name='MyTest.test2']

```

Indicates that the `testname` test failed. Only one `testFailed` message can appear for a given test name.
* `message` contains the textual representation of the error.
* `details` contains detailed information on the test failure, typically a message and an exception stacktrace.
* `actual` and `expected` attributes can only be used together with `type='comparisonFailure'` to report comparison failure. The values will be used when opening the test in the IDE.

Here is a longer example of test reporting with service messages:

```Shell
##teamcity[testSuiteStarted name='suiteName']
##teamcity[testSuiteStarted name='nestedSuiteName']
##teamcity[testStarted name='package_or_namespace.ClassName.TestName']
##teamcity[testFailed name='package_or_namespace.ClassName.TestName' message='The number must be 20000' details='junit.framework.AssertionFailedError: expected:<20000> but was:<10000>|n|r    at junit.framework.Assert.fail(Assert.java:47)|n|r    at junit.framework.Assert.failNotEquals(Assert.java:280)|n|r...']
##teamcity[testFinished name='package_or_namespace.ClassName.TestName']
##teamcity[testSuiteFinished name='nestedSuiteName']
##teamcity[testSuiteFinished name='suiteName']

```

#### Enabling Test Retry

You can enable the support of _test retry_ for a build configuration. With this option enabled, the successful run of a test will mute its previous failure, which means that TeamCity will mute a test if it fails and then succeeds within the same build.   
Such tests will not affect the build status.

```Shell
##teamcity[testRetrySupport enabled=’true’]
```

#### Interpreting Test Names

A full test name can have a form of: `<suite name>: <package/namespace name>.<class name>.<test method>(<test parameters>)`, where `<class name>` and `<test method>` cannot have dots in the names. Only `<test method>` is a mandatory part of a full test name.

The full test name is used to compare tests between consequent builds or match tests across different build configurations.

Example of a full test name:

```JavaScript
Integration Tests: Backend: org.jetbrains.teamcity.LoginPageController.testBadPassword("incorrect password", false)
 
// in the example above, 
// suite name = "Integration Tests: Backend"
// package = org.jetbrains.teamcity
// class name = LoginPageController
// test method = testBadPassword
// test parameters = ("incorrect password", false)
```

The __[Tests](build-results-page.md#Tests+Tab)__ tab of the __Build Results__ page allows grouping by suites, packages/namespaces, classes, and tests. Usually the attribute values are provides as they are reported by your test framework and TeamCity is able to interpret test names correctly.

If a test cannot be parsed in the form above, TeamCity still tries to extract `<suite name>` from the full test name for the filtering on the Tests tab, and treats everything after the suite a non-parsable test name.

#### Reporting Additional Test Data

It is possible to attach extra information to the tests using the `testMetadata` service message. More details are available on a [separate page](reporting-test-metadata.md).

### Reporting .NET Code Coverage Results

You can configure .NET coverage processing by means of service messages. To learn more, refer to [Manually Configuring Reporting Coverage](manually-configuring-reporting-coverage.md) page.

### Reporting Inspections

You can report inspections from a custom tool to TeamCity using the service messages described below.

Among other uses, the number of inspections can be used as a build metric to [fail a build on](build-failure-conditions.md#Fail+Build+on+Metric+Change).

#### Inspection Type

Each specific warning or an error in code (inspection instance) has an inspection type — the unique description of the conducted inspection, which can be reported via


```Shell
##teamcity[inspectionType id='<id>' name='<name>' description='<description>' category='<category>']

```

where all the attributes are required and can have either numeric or textual values:

* `id` — (mandatory) limited by 255 characters
* `name` — (mandatory) limited by 255 characters
* `category` — (mandatory) limited by 255 characters. The `category` attribute examples are "Style violations" and "Calling contracts".
* `description` — (mandatory) limited by 4000 characters. The description can also be in HTML. For example,

```HTML
<html>
    <body>
    Reports unnecessary local variables, which add nothing to the comprehensibility of a method. Variables caught include local variables which are immediately returned, local variables that are immediately assigned to another variable and then not used, and local variables which always have the same value as another
    local variable or parameter.
    <!-- tooltip end -->
        <p>
        Use the first checkbox below to have this inspection ignore variables which are immediately returned or thrown. Some coding styles suggest using such variables for clarity and ease of debugging.
        </p>
        <p>
        Use the second checkbox below to have this inspection ignore variables which are annotated.
        </p>
    </body>
</html>

```

### Inspection Instance

Reports a specific defect, warning, error message. Includes location, description, and various optional and custom attributes.

```Shell
##teamcity[inspection typeId='<inspection type identity>' message='<instance description>' file='<file path>' line='<line>' additional attribute='<additional attribute>']

```

where all the attributes can have either numeric or textual values:
* `typeId` — (mandatory), reference to the `inspectionType.id` described [above](#Inspection+Type) limited by 255 characters.
* `message` — (optional) current instance description limited by 4000 characters.
* `file` — (mandatory) file path limited by 4000 characters. The path can be absolute or relative to the [checkout directory](build-checkout-directory.md).
* `line` — (optional) line of the file, integer.
* `additional attribute`– can be any attribute, `SEVERITY` is often used here, with one of the following values (mind the upper case): `INFO`, `ERROR`, `WARNING`, `WEAK WARNING`.

Example:

```Shell
##teamcity[inspectionType id='UnnecessaryLocalVariable' name='Redundant local variable' description='<html><body>Reports unnecessary local variables...</body>
</html>' category='Data flow issues']
##teamcity[inspection typeId='UnnecessaryLocalVariable' message='Local variable <code>i</code> is redundant' file='src/Test.java' line='19' SEVERITY='WARNING']

```

### Publishing Artifacts While Build is in Progress

You can publish the build artifacts while the build is still running, immediately after the artifacts are built.

To do this, you need to output the following line:

```Shell
##teamcity[publishArtifacts '<path>']

```

The `<path>` has to adhere to the same rules as the [Build Artifact specification](configuring-general-settings.md#Artifact+Paths) of the __Build Configuration Settings__. The files matching the `<path>` will be uploaded and visible as the artifacts of the running build.

The message should be printed after all the files are ready and no file is locked for reading.

>To publish multiple artifact files in one archive, you need to configure the _[Artifact paths](configuring-general-settings.md#Artifact+Paths)_ in __General Settings__ of a build configuration. If you use service messages, only artifacts for the last rule will be published to the archive.
>
{type="tip"}

Artifacts are uploaded in the background, which can take time. Make sure the matching files are not deleted till the end of the build (for example, you can put them in a directory that is cleaned on the next build start, in a [temp directory](how-to.md#Make+Temporary+Build+Files+Erased+between+the+Builds), or use [Swabra](build-files-cleaner-swabra.md) to clean them after the build).

<note>

The process of publishing artifacts can affect the build, because it consumes network traffic, and some disk/CPU resources (should be pretty negligible for not large files/directories).

</note>

Artifacts that are specified in the build configuration setting will be published as usual.

### Passing NuGet Packages Between Steps

If you need to publish NuGet packages and then use their contents within one build, you want to guarantee they are published and indexed on time — and not at the build finish.   
For this, you can use a [NuGet Publish](nuget-publish.md) runner or send the `##teamcity[publishNuGetPackage]` service message in any step instead. This ensures the NuGet packages are published in all configured NuGet feeds right at the end of the current step and are available in the following build steps.

### Reporting Build Progress

You can use special progress messages to mark long-running parts in a build script. These messages will be shown on the projects' dashboard for the corresponding build and on the __[Build Results](working-with-build-results.md)__ page.

To log a single progress message, use:

```Shell
##teamcity[progressMessage '<message>']

```

This progress message will be shown until another progress message occurs or until the next target starts (in case of Ant builds).

If you wish to show a progress message for a part of a build only, use:

```Shell
##teamcity[progressStart '<message>']
...some build activity...
##teamcity[progressFinish '<message>']

```

<note>

The same message should be used for both `progressStart` and `progressFinish`. This allows nesting progress blocks. Note that in case of Ant builds, progress messages will be replaced if an Ant target starts.
</note>

### Reporting Build Problems

To fail a build directly from the build script, a build problem must be reported. Build problems affect the build status text. They appear on the __[Build Results](working-with-build-results.md)__ page. To add a build problem to a build, use:


```Shell
##teamcity[buildProblem description='<description>' identity='<identity>']

```

where:
 *  `description` (mandatory): a human-readable plain text describing the build problem. By default, the `description` appears in the build status text and in the list of build's problems. The text is limited to 4000 symbols, and will be truncated if the limit is exceeded.
 * `identity` (optional): a unique problem ID. Different problems must have different identity, same problems — same identity, which should not change throughout builds if the same problem, for example, the same compilation error occurs. It must be a valid Java ID up to 60 characters. If omitted, the `identity` is calculated based on the `description` text.

[//]: # (Internal note. Do not delete. "Build Script Interaction with TeamCityd44e948.txt")    

### Reporting Build Status

TeamCity allows changing the __build status text__ from the build script. Unlike [progress messages](#Reporting+Build+Progress), this change persists even after a build has finished.  
You can also change the build status of a failing build to `SUCCESS`.

>To change the build status to `FAILURE`, uses the [`buildProblem`](#Reporting+Build+Problems) service message.

To set the status and/or change the text of the build status (for example, note the number of failed tests if the test framework is not supported by TeamCity), use the `buildStatus` message with the following format:

```Shell
##teamcity[buildStatus status='<status_value>' text='{build.status.text} and some aftertext']

```

where:
 * `status` (optional): use the `SUCCESS` value to change the build status to _Success_.
 * `text` (mandatory): set the new build status text.    
 Optionally, the text can use the `{build.status.text}` substitution pattern, which represents the status calculated by TeamCity automatically using passed test count, compilation messages, and so on.   
The status set will be presented while the build is running and will also affect the final build results.

### Reporting Build Number

To set a custom build number directly, specify a `buildNumber` message using the following format:

```Shell
##teamcity[buildNumber '<new build number>']

```

In the &lt;new build number&gt; value, you can use the `{build.number}` substitution to use the current build number automatically generated by TeamCity. For example:

```Shell
##teamcity[buildNumber '1.2.3_{build.number}-ent']

```

### Adding or Changing Build Parameter
{id='set-parameter'}

By using a dedicated service message in your build script, you can dynamically update build parameters of the build right from a build step (the parameters need to be defined in the __[Parameters](configuring-build-parameters.md)__ section of the build configuration). The changed build parameters will be available in the build steps following the modifying one. They will also be available as build parameters and can be used in the dependent builds via [` %dep.*% parameter references`](predefined-build-parameters.md#Dependency+Parameters), for example:

```Shell
##teamcity[setParameter name='ddd' value='fff']
```

When specifying a build parameter's name, mind the prefix:
* `system` for system properties.
* `env` for environment variables.
* No prefix for configuration parameters.

[Read more](configuring-build-parameters.md) about build parameters and their prefixes.

>Since the `setParameter` mechanism does not publish anything to the server until the build is finished, it is not possible to get updated parameters during the build via the REST API.
>
{type="note"}

### Reporting Build Statistics

In TeamCity, it is possible to configure a build script to report statistical data and then display the charts based on the data. Refer to the [Customizing Statistics Charts](customizing-statistics-charts.md#Modifying+Predefined+Project-level+Charts) page for a guide to displaying the charts on the web UI. This section describes how to report the statistical data from the build script via service messages. You can publish the build statics values in two ways:
* Using a service message in a build script directly
* [Providing data using the teamcity-info.xml file](teamcity-info-xml.md#Storing+Data+in+teamcity-info.xml)   
To report build statistics using service messages: Specify a `buildStatisticValue` service message with the following format for each statistics value you want to report:

```Shell
##teamcity[buildStatisticValue key='<valueTypeKey>' value='<value>']

```

where
* `key` must not be equal to any of [predefined keys](custom-chart.md#Default+Statistics+Values+Provided+by+TeamCity).
* `value` must be a positive/negative integer of up to 13 digits; float values with up to 6 decimal places are also supported.

### Disabling Service Messages Processing

If you need for some reason to disable searching for service messages in the output, you can disable the service messages search with the messages:

```Shell
##teamcity[enableServiceMessages]
##teamcity[disableServiceMessages]

```

Any messages that appear between these two are not parsed as service messages and are effectively ignored. For server-side processing of service messages, enable/disable service messages also supports the `flowId` attribute and will ignore only the messages with the same `flowId`.

[//]: # (Internal note. Do not delete. "Build Script Interaction with TeamCityd44e1141.txt")    

### Importing XML Reports

In addition to the [UI build feature](xml-report-processing.md), XML reporting can be configured from within the build script with the help of the `importData` service message. Also, the message supports importing of the previously collected code coverage and code inspection/duplicates reports.

The service message format is:

```Shell
##teamcity[importData type='typeID' path='<path to the xml file>']

```

<note>

To be processed, report XML files (or a directory) must be located in the [checkout directory](build-checkout-directory.md), and the path must be relative to this directory.

</note>

where `typeID` can be one of the following (see also [XML Report Processing](xml-report-processing.md)):

<table>

<tr>

<td>

typeID

</td>

<td>

Description

</td>

</tr><tr>

<td colspan="2">

__Testing frameworks__

</td>

</tr><tr>

<td>

`junit`

</td>

<td>

JUnit Ant task XML reports

</td></tr><tr>

<td>

`surefire`

</td>

<td>

Maven Surefire XML reports

</td></tr>

<tr>

<td>

`nunit`

</td>

<td>

NUnit-Console XML reports

</td></tr>

<tr>

<td>

`mstest`

</td>

<td>

MSTest XML reports

</td></tr>

<tr>

<td>

`vstest`

</td>

<td>

VSTest XML reports

</td></tr>

<tr>

<td>

`gtest`

</td>

<td>

Google Test XML reports

</td></tr>

<tr>

<td colspan="2">

__Code inspection__

</td></tr>

<tr>

<td>

`intellij-inspections`

</td>

<td>

IntelliJ IDEA inspection results

</td></tr>

<tr>

<td>

`checkstyle`

</td>

<td>

Checkstyle inspections XML reports

</td></tr>

<tr>

<td>

`findBugs` <sup>2)</sup>

</td>

<td>

FindBugs inspections XML reports

</td></tr>

<tr>

<td>

`jslint`

</td>

<td>

JSLint XML reports

</td></tr>

<tr>

<td>

`ReSharperInspectCode` <sup>1)</sup>

</td>

<td>

ReSharper `inspectCode.exe` XML reports

</td></tr>

<tr>

<td>

`FxCop`<sup>1</sup>

</td>

<td>

FxCop inspection XML reports

</td></tr>

<tr>

<td>

`pmd`


</td>

<td>

PMD inspections XML reports

</td></tr>

<tr>

<td colspan="2">

__Code duplication__

</td></tr>

<tr>

<td>

`pmdCpd`

</td>

<td>

PMD Copy/Paste Detector (CPD) XML reports

</td></tr>

<tr>

<td>

`DotNetDupFinder` <sup>1)</sup>

</td>

<td>

ReSharper `dupfinder.exe` XML reports

</td></tr>

<tr>

<td colspan="2">

__Code coverage__

</td></tr>

<tr>

<td>

`dotNetCoverage` <sup>1) 3)</sup>

</td>

<td>

XML reports generated by dotcover, partcover, ncover or ncover3

</td>
</tr>
</table>

Notes:
1) Only supports a specific file in the `path` attribute.
2) Also requires the `findBugsHome` attribute specified pointing to the home directory of the installed FindBugs tool.
3) Also requires the `tool='<tool name>'` service message attribute, where the `<tool name>` is either `dotcover`, `partcover`, `ncover` or `ncover3`.

If not specially noted, the report types support Ant-like wildcards in the `path` attribute.

* `verbose='true'` attribute will enable detailed logging into the build log.
* `parseOutOfDate='true'` attribute will process all the files matching the path. By default, only those updated during the build (determined by last modification timestamp) are processed. If any not matching reports are found, the "report skipped as out-of-date" message appears in the build log.
* `whenNoDataPublished=<action>` (where `<action>` is one of the following: `info` (default), `nothing`, `warning`, `error`) will change output level if no reports matching the path specified were found.
* (__deprecated, use [Build Failure Conditions](build-failure-conditions.md) instead__)   
  `findBugs`, `pmd` or `checkstyle` importData messages also take optional `errorLimit` and `warningLimit` attributes specifying errors and warnings limits, exceeding which will cause the build failure.

<note>

After the `importData` message is received, TeamCity agent starts to monitor the specified paths on the disk and imports matching report files in the background as soon as the files appear on disk.   
The parsing only occurs within the build step in which the messages were received. On the step finish, the agent ensures all the present reports are processed before beginning the next step. This behavior is different from that of [XML Report Processing](xml-report-processing.md) build feature, which completes files parsing only at the end of the build.   
Ensure the report files are available after the generation process ends (the files are not deleted, nor overwritten by the build script)

</note>

To initiate monitoring of several directories or parse several types of the report, send the corresponding service messages one after another.

<note>

Only several reports of different types can be included in a build. Processing reports of several inspections or duplicates tools in a single build is not supported. See the [related feature request](http://youtrack.jetbrains.com/issue/TW-14260).

</note>

[//]: # (Internal note. Do not delete. "Build Script Interaction with TeamCityd44e1503.txt")    

## Canceling Build via Service Message

If you need to cancel a build from a script, for example, if a build cannot proceed normally due to the environment, or a build should be canceled form a subprocess, you can use the following service message:

```Shell
echo "##teamcity[buildStop comment='canceling comment' readdToQueue='true']"
```

If required, you can re-add the build to the queue after canceling it. By default, TeamCity will do 3 attempts to re-add the build into the queue. 

## Libraries Reporting Results via TeamCity Service Messages

Several platform-specific libraries from JetBrains and external sources are able to report the results  via TeamCity Service messages.
* [Service messages .NET library](https://github.com/JetBrains/TeamCity.ServiceMessages) — .NET library for generating (and parsing) TeamCity service messages from .NET applications. See a [related blog post](http://blog.jetbrains.com/teamcity/2011/10/teamcity-service-messages-library-for-net/).
* [Jasmine 2.0 TeamCity reporter](https://github.com/WilliamDoman/Jasmine2.0TeamCityReporter) — support for emitting TeamCity service messages from Jasmine 2.0 reporter
* [Perl TAP Formatter](http://search.cpan.org/~thaljef/TAP-Formatter-TeamCity-0.04/) — formatter for Perl to transform TAP messages to TeamCity service messages
* [PHPUnit 5.0](https://github.com/sebastianbergmann/phpunit/blob/9e86c85be3302eb125f15037ae6f496f62750a93/ChangeLog-5.0.md#500---2015-10-02) — supports TeamCity service messages for tests. For earlier PHPUnit versions, the following external libraries can be used: [PHPUnit Listener 1](https://github.com/realweb-team/deploytools), [PHPUnit Listener 2](https://github.com/maartenba/phpunit-runner-teamcity) — listeners which can be plugged via PHPUnit's` suite.xml` to produce TeamCity service messages for tests.
* [Python Unit Test Reporting to TeamCity](https://pypi.python.org/pypi/teamcity-messages) — the package that automatically reports unit tests to the TeamCity server via service messages (when run under TeamCity and provided the testing code is adapted to use it).
* [Mocha](https://github.com/visionmedia/mocha) — on-the-fly reporting via service messages for Mocha JavaScript testing framework. See the related [post](http://richarddingwall.name/2012/06/17/running-mocha-browser-tests-in-teamcity/) with instructions.
* [Karma](https://github.com/karma-runner/karma) — support in the JavaScript testing tool to report tests progress into TeamCity using TeamCity service messages
