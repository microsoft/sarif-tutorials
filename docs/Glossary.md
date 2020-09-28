[Table of contents](../README.md#contents)

# Appendix: Glossary

<a id="analysis-tool"></a>**analysis tool**: A tool that produces observations about a set of programming
<a href="#artifact">_artifacts_</a>, for example by detecting <a href="#issue">_issues_</a>
or calculating metrics.

<a id="artifact"></a>**artifact**: Anything produced by the act of programming, such as a source file, an object file,
or a web page.

<a id="binding"></a>**binding**: An association between two sets of entities.
In this context, the association between the entities defined by the SARIF <a href="#object-model">_object model_</a>
and their representation in a programming language.

<a id="code-flow"></a>**code flow**: A set of one or more <a href="#thread-flow">_thread flows_</a>
which together specify a pattern of code execution relevant to detecting a <a href="#result">_result_</a>.

<a id="custom-taxonomy"></a>**custom taxonomy**: A <a href="#taxonomy">_taxonomy_</a>
defined by an analysis tool.

<a id="driver"></a>**driver**: That <a href="#tool-component">_component_</a> of an
<a href="#analysis-tool">_analysis tool_</a> which contains the tool's
<a href="#primary-executable">_primary executable_</a>.

<a id="dynamic-analysis-tool"></a>**dynamic analysis tool**: An <a href="#analysis-tool">_analysis tool_</a> that
observes the execution of a program.

<a id="embedded-link"></a>**embedded link**: A hyperlink that occurs in a plain text message or a Markdown message.

<a id="engineering-system"></a>**engineering system**: A software development environment within which
<a href="#analysis-tool">_analysis tools_</a> execute.
It typically includes a build system, a source control system,
a <a href="#result-management-system">_result management system_</a>,
a bug tracking system, a test execution system, and so on.

<a id="extension"></a>**extension**: A <a href="#tool-component">_component_</a> of an
<a href="#analysis-tool">_analysis tool_</a> other than the <a href="#driver">_driver_</a>.
Extensions are typically authored separately from and discovered dynamically by the driver.
They often contain additional <a href="#rule">_analysis rules_</a>.

<a id="external-property-file"></a>**external property file**: A file separate from a SARIF log file,
typically used to store large data sets (such as <a href="#taxonomy">_taxonomies_</a>) needed by multiple log files.

<a id="first-class-property"></a>**first-class property**: A property defined by the SARIF specification, as opposed to
one that occurs in a <a href="#property-bag">_property bag_</a>.

<a id="hierarchical-string"></a>**hierarchical string**: A string in which the forward slash character `'/'`
is significant and defines a logical hierarchy on the values of the string.

<a id="issue"></a>**issue**: A condition in a program that might detract from its quality.

<a id="logical-location"></a>**logical location**: A location specified by name, without reference to a particular
<a href="artifact">_artifact_</a>, for example, by means of a class name and a method name.

<a id="notification"></a>**notification**: A message from an <a href="#analysis-tool">_analysis tool_</a>
that provides information about the tool's configuration or execution.

<a id="notification-metadata"></a>**notification metadata**: Information that describes
a <a href="#notification">_notification_</a> produced by
an <a href="#analysis-tool">_analysis tool_</a>.
SARIF uses the same object (`reportingDescriptor`) to describe both notification metadata and
<a href="#rule-metadata">_rule metadata_</a>.

<a id="object-model"></a>**object model**: A set of classes and associations among them that describes a problem domain.
In this context, the set of classes that describe the contents of a SARIF log file.

<a id="physical-location"></a>**physical location**: A location specified by reference to a particular
<a href="#artifact">_artifact_</a>, for example, by means of a file name and a line number.

<a id="post-processor"></a>**post-processor**: A program that takes a SARIF file, modifies it,
and produces a new SARIF file with the modifications.

<a id="primary-executable"></a>**primary executable**: That file belonging to an
<a href="#analysis-tool">_analysis tool_</a> in which execution begins,
for example, a binary file containing the program entry point.

<a id="property-bag"></a>**property bag**: A SARIF property named `properties` that can contain any property
with any value.

<a id="result"></a>**result**: An observation about an artifact, often but not always an <a href="#issue">_issue_</a>.

<a id="result-management-system"></a>**result management system**: A system that consumes
<a href="#result">_results_</a> produced by <a href="#analysis-tool">_analysis tools_</a >
and typically performs functions such as filing bugs
and producing reports that provide a view of system quality over time.

<a id="related-location"></a>**related location**: A place in the code other than the
<a href="#result-location">_result location_</a> that helps a user understand a <a href="#result">_result_</a>.

<a id="result-location"></a>**result location**: A place in the code where an <a href="#analysis-tool">_analysis tool_</a>
detects a <a href="#result">_result_</a>.

<a id="rule"></a>**rule**: A criterion for correctness verified by an <a href="#analysis-tool">_analysis tool_</a>.

<a id="rule-metadata"></a>**rule metadata**: Information that describes a <a href="#rule">_rule_</a> supported by
an <a href="#analysis-tool">_analysis tool_</a>.
SARIF uses the same object (`reportingDescriptor`) to describe both rule metadata and
<a href="#notification-metadata">_notification metadata_</a>.

<a id="standard-taxonomy"></a>**standard taxonomy**: A <a href="#taxonomy">_taxonomy_</a>
defined publicly, without reference to any particular analysis tool.

<a id="static-analysis-tool"></a>**static analysis tool**: An <a href="#analysis-tool">_analysis tool_</a>
that examines programming <a href="#artifact">_artifacts_</a> without executing the program.

<a id="taxon"></a>**taxon** (_pl._ **taxa**): An individual category within a <a href="#taxonomy">_taxonomy_</a>.

<a id="taxonomy"></a>**taxonomy**: A system that classifies <a href="#result">_results_</a> into a set of categories.

<a id="thread-flow"></a>**thread flow**: A temporally ordered set of code locations
specifying a possible execution path through the code,
which occur within a single thread of execution, such as an operating system thread or a fiber.

<a id="tool-component"></a>**tool component**: A component of an <a href="#analysis-tool">_analysis tool_</a>,
either its <a href="#driver">_driver_</a> or an <a href="#extension">_extension_</a>,
consisting of one or more files.

<a id="config-notif"></a>**tool configuration notification**: A <a href="#notification">_notification_</a>
that provides information about how the tool was configured,
for example, what options were selected or which <a href="#rule">_rules_</a> were enabled and disabled.

<a id="exec-notif"></a>**tool execution notification**: A <a href="#notification">_notification_</a>
that provides information about runtime conditions encountered during the tool's execution,
such as the analysis start and end times, or an exception encountered during the evaluation of
a <a href="#rule">_rule_</a>.

[Table of contents](../README.md#contents)
