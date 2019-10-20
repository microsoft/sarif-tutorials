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

TODO: Required properties; common properties; security and redaction.

## <a id="notifications"></a>Notifications

## <a id="taxonomies"></a>Taxonomies

## <a id="code-flows"></a>Code flows

## Notes

<a id="note-1"></a>1. See
[§3.14.11, invocations property](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012451)

[Table of contents](../README.md#contents)
