[Table of contents](../README.md#contents)

# Appendix: Fitness for purpose: Automatic bug filing

## Introduction

This Appendix describes how to produce SARIF log files that are <a href="Glossary.md#fit-for-purpose">_fit for purpose_</a> for automatic bug filing. It is an example of the "playbook" for producing fit-for-purpose SARIF log files, described in the Appendix [Fitness for purpose: Overview](Fitness-for-purpose-overview.md).

The standard described here works well for a particular bug filing system used internally at Microsoft.
You might be able to use it without change, or you might have to adapt it to your company's engineering system.
In that case, do read the [Overview](Fitness-for-purpose-overview.md) to understand the purpose of each component of the playbook given here.

## The standard

For automatic bug filing to be effective, the engineering system must compare the SARIF results from the current build to those from a previous build, to identify those results that are new in the current build.
This process is known as <a href="Glossary.md#result-matching">_result matching_</a>.
Some of the requirements of this standard help to ensure that result matching is accurate, reducing the number of duplicate bugs that are filed (and ensuring that bugs _are_ filed for truly new results).
Other requirements ensure that the filed bugs contain enough information to be understood and acted upon.

Every requirement in the the following list is a "must", meaning that the validator must produce an `"error"`-level message for all of them, and the bug filing system must refuse to accept a file that violates any of them.

1. Result `message` objects must provide `id` and `arguments` (as opposed to only providing `text`).
    The result matching algorithm (implemented in the [SARIF Multitool's](Multitool.md) `match-results-forward` command) includes `id` and `arguments` in its matching criteria.

2. The `run` object must provide `versionControlProvenance`.
    The bug filing system uses this information (which includes the URL of the repo containing the files that were analyzed) to decide which team should be assigned the filed bugs.

3. The `run.tool.driver` object must provide at least one of `version`, `semanticVersion`, or `dottedQuadFileVersion`.
    The bug filing system uses this information to decide whether the current run was analyzed with a different tool version than the previous run.
    If the tool version has changed, the bug filing system must re-analyze the sources as they existed during the previous run with the new version of the tool.
    This prevents a discontinuity due to any new analysis rules that the new tool version might have introduced.
    The bug filing system will only file bugs for results introduced since the last run -- whether those results come from previously existing or newly introduced rules.

4. The `run.tool.driver` object must provide `informationUri`. This helps the developer responsible for fixing the bug by providing a way to learn more about the tool.

5. All result location URIs must be expressed as relative references with respect to the repo root.

    **TODO** Understand why it's not good enough to have absolute URIs if they start with repo root (as specified by `versionControlDetails.mappedTo`).

6. Each rule must provide `helpUri`. This helps the developer responsible for fixing the bug by providing a way to learn more about the rule violation.

7. Each result must enable the user to view the problem in context.
    The result must include a code snippet for a region containing a few lines on either side of the actual result (`result.locations[].physicalLocation.contextRegion.snippet`).
    It is not enough to provide `...physicalLocation.region.snippet`: the region designating the actual error might be only a few characters long, or even just an insertion point.

    In addition, the log file must embed the entire contents of every file in which the tool detected a result so that the appropriate file can be attached to each filed bug.
    This is important for two reasons.
    First, the user assessing the bug might not yet have an enlistment, and if they do, they might not have checked out the branch that was analyzed.
    In some source control systems, switching branches on a large code base can be time consuming.
    Second, the user assessing the bug might not have permission to enlist. For example, the user might be a security analyst in an organization where only developers can enlist.

## The analysis rules

The analysis rules that enforce these standards are:

- `SARIF2002.ProvideMessageArguments`: Enforces Requirement <span>#</span>1 by ensuring that `result.message.id` and `.arguments` are present.
- `SARIF2003.ProvideVersionControlProvenance`: Enforces Requirement <span>#</span>2 by ensuring that `run.versionControlProvenance` is present.
- `SARIF2005.ProvideToolProperties`: Enforces Requirement <span>#</span>3 by ensuring that at least one of `version`, `semanticVersion`, and `dottedQuadFileVersion` is present, and <span>#</span>4 by ensuring that `informationUri` is present.<sup><a href="#note-1">1</a></sup>
- `SARIF2007.ExpressPathsRelativeToRepoRoot`: Enforces Requirement <span>#</span>5 by ensuring that all result location URIs are expressed as relative references with respect to the repo root.
- `SARIF2012.ProvideHelpUris`: Enforces Requirement <span>#</span>6 by ensuring that `helpUri` is present.
- `SARIF2010.ProvideSupplementalCodeContext`: Enforces Requirement <span>#</span>7 by ensuring that context region snippets and embedded file content are present.<sup><a href="#note-2">2</a></sup>

## The fixers

The following procedure will address requirements <span>#</span>2 (provide `run.versionControlProvenance`), <span>#</span>5 (express URIs relative to repo roots), and <span>#</span>7 (embed file contents, and provide code snippets for context regions).
To embed file content and provide code snippets, the code we describe here must be run from the root directory of the repository.

There is no way to automatically enforce the rest of the criteria (for example, there's no way to provide a tool information URI if the tool itself didn't provide one). Also, this procedure will only provide code snippets for context regions if the tool itself provided context regions.

1. In whatever program you are using to prepare SARIF for bug filing, add a NuGet reference to `Sarif.Multitool.Library`, version 2.3.5 or later.

2. Write the following code:

```C#
var command = new RewriteCommand();
var options = new RewriteOptions
{
    InputFilePath = "<input-file>",
    OutputFilePath = "<output-file>",
    DataToInsert = new List<OptionallyEmittedData>
    {
        OptionallyEmittedData.ContextRegionSnippets,
        OptionallyEmittedData.TextFiles,
        OptionallyEmittedData.BinaryFiles,
        OptionallyEmittedData.VersionControlInformation
    }
};

int exitCode = command.Run(options);
```

This is equivalent to the SARIF Multitool command line

```
sarif rewrite --insert ContextRegionSnippets,TextFiles,BinaryFiles,VersionControlInformation --output <output-file> input-file
```

## The configuration file

This is a standard validation XML configuration file, which explicitly enables all the analysis rules mentioned above, and sets their levels as `"error"`.

## Notes

<a id="note-1">1.</a> At the time of writing, `SARIF2005` is not satisfied by `dottedQuadFileVersion`, and it does not require `informationUri`. Issue [microsoft/sarif-sdk#2040](https://github.com/microsoft/sarif-sdk/issues/2040), "SARIF2005.ProvideToolProperties: Allow dottedQuadFileVersion; require informationUri", tracks this work.

<a id="note-2">2.</a> At the time of writing, `SARIF2010` covers code snippets, `SARIF2011` covers context regions, and `SARIF2013` covers embedded file content. Issue [microsoft/sarif-sdk#2041](https://github.com/microsoft/sarif-sdk/issues/2041), "Combine 'code context' rules", tracks this work.

<a id="note-3">3.</a> This should probably be just an option `--rebase-uris` to the `rewrite` command.

[Table of contents](../README.md#contents)