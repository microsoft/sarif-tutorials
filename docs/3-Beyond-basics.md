[Table of contents](../README.md#contents)

# Beyond the Basics

## <a id="more-about-messages"></a>More about messages

## <a id="invocations"></a>Invocations

We have seen that a `run` object describes a single invocation of a single analysis tool.
We've seen that the required `run.tool` property describes the tool that produced the run.
Now we'll discuss the optional `run.invocations` property, which describes how the tool was invoked.

`run.invocations` is an _array_ of `invocation` objects.
Since a `run` describes a single invocation, why is `run.invocations` an array?
The spec explains:<sup><a href="#note-1">1</a></sup>

> A `run` object **MAY** contain a property named `invocations` whose value is an array of zero or more
> `invocation` objects (§3.20) that together describe a single run of a single analysis tool.
>
> **NOTE**: Normally, an analysis tool runs as a single process, and the `invocations` array requires only one element.
> The `invocations` property is defined as an array, rather than as a single `invocation` object,
> to accommodate tools which execute a sequence of programs to produce results.
> For example, a tool might run one program to determine the set of artifacts to analyze and another program
> to analyze those artifacts.
> The elements of the invocations array **SHOULD**, as far as possible, be arranged in chronological order according to
> the start time of each process. If some of the processes run in parallel, this might not be possible.

Note that the spec clearly states that the elements of the array together describe a single run of a single tool.
It's not correct for it to contain (for example) an invocation of Clang Static Analyzer followed by
an invocation of ESLint,
or two successive invocations of Clang Static Analyzer.

The only required property of the `invocation` object is the Boolean `executionSuccessful`.
It's required because you can't tell from the integer `exitCode` property whether the tool succeeded or failed:
not every tool returns 0 on success and non-zero on failure.

There are properties to capture the command line, both as a single string (`commandLine`) and parsed into
arguments (`arguments`).
There are properties for the start and end time (`startTimeUtc` and `endTimeUtc`<sup><a href="#note-2">2</a></sup>),
for machine and environment information (`machine`, `account`, `processId`, `workingDirectory`,
`environmentVariables`),
and to capture the standard IO streams (`stdin`, `stdout`, `stderr`, `stdoutStderr`).

Most important, there are properties to capture "notifications" produced by the tool.
We'll discuss those next.

If you capture the command line, or if you use the environment-related properties like `machine`, `account`,
and `environmentVaribles`, be aware that they can contain sensitive information.
SARIF offers a facility to "redact" sensitive information, and you should become familiar with it.<sup><a href="#note-3">3</a></sup>

## <a id="notifications"></a>Notifications

In addition to analysis results, many tools provide information about their execution,
such as progress notifications (for example, "Execution started." or "Analysing directory src/io...")
and error conditions (for example, "Rule CA1304 threw an exception and has been disabled." or
"Rule CA9999 cannot be enabled because it does not exist.").

### <a id="exec-config-notif"></a>Tool execution notification and tool configuration notifications

SARIF distinguishes two types of tool notifications,
<a href="5.2-Glossary.md#config-notif">_tool configuration notifications_</a> and
<a href="5.2-Glossary.md#exec-notif">_tool execution notifications_</a>.

Tool configuration notifications provide information about the configuration of the tool,
for example, which options were selected, or which rules were enabled or disabled.
They are found in the optional `invocation.toolConfigurationNotifications` property.

Tool execution notifications provide information about runtime conditions encountered
during the tool's execution,
such as the analysis start and end times, or an exception encountered during the evaluation of a rule.
They are found in the optional `invocation.toolExecutionNotifications` property.

Both `toolConfigurationNotifications` and `toolExecutionNotifications` are arrays of `notification`
objects.

Part of the rationale for distinguishing these notification types is that they are of interest to different audiences.
Tool configuration notifications are of interest to build engineers;
tool execution notifications (especially those that report runtime exceptions) are of interest to tool authors.
Of course both might be of interest to tool users, and on smaller teams the same people might fill all of these roles.

### <a id="notif-result"></a>Notifications _vs._ results

Notifications and results have much in common:

- They can both have a severity level (for example, "Analysis started."
is a `"note"`-level notification, which "Rule CA1304 threw an exception." is an `"error"`-level notification.)
- They both have user-facing messages, possibly parameterized (for example, `"Rule {0} threw an exception."`).
- Both can be described by additional metadata such as a full description, a help URI, and so forth.

For this reason, SARIF uses the same object to describe both rule metadata
and what the spec refers to as <a href="5.2-Glossary.md#notification_metadata">_notification metadata_</a>:
the `reportingDescriptor`.<sup><a href="#note-4">4</a></sup>

Note that SARIF does _not_ use the same object to represent the results and notifications themselves:
a `result` object is not the same as a `notification` object.
This is because there are so many properties of a `result` (for example, `codeFlows`) that's don't apply to
notifications.

So in our [simple example](1-Introduction.md#simple-example-file),
the property `tool.driver.rules` was actually an array of `reportingDescriptor`s,
and `tool.driver` has an additional property `notifications` that is also an array of `reportingDescriptor`s.

## <a id="taxonomies"></a>Taxonomies

## <a id="code-flows"></a>Code flows

## <a id="automation"></a>Automation

## Notes

<a id="note-1"></a>1. See
[§3.14.11, invocations property](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012451)

<a id="note-2"></a>2. All times in SARIF's first-class properties are expressed in UTC, and the properties are named
to remind you of that.

<a id="note-3"></a>3. See
[§3.5.2, Redactable strings](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012393),
[§3.14.28, redactionTokens property](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012468),
and search for the string "redactable" in the spec to find all the tokens that might contain sensitive information.

<a id="note-4"></a>4. Before Michael Fanning noticed the similarities between results and notifications,
this object was called simply the `rule` object.
This is a case where in my opinion the generalization of a concept led to a name that was less understandable.
But despite my reputation for being a good "namer," I've never been able to come up with a better one.

[Table of contents](../README.md#contents)
