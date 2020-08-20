[Table of contents](../README.md#contents)

# Appendix: Fitness for purpose: An overview

## Introduction

This Appendix defines a "playbook" for ensuring that a SARIF log file contains the information it needs so it can be used for any particular purpose. We refer to a SARIF log file that contains the necessary information as being <a href="Glossary.md#fit-for-purpose">_fit for purpose_</a>.

SARIF log files can be used in many ways. For example:

- **Viewing**: An end user can examine a log file in a SARIF "viewer", such as this [web-based viewer](https://microsoft.github.io/sarif-web-component/).

- **Result matching**: A build system can run static analysis tools as part of a nightly build.
It can compare the SARIF logs from the current build with corresponding logs from the previous build, and notify the team of any new results. We refer to the process of identifying results that have appeared, disappeared, or changed from one run to another as <a href="Glossary.md#result-matching">_result matching_</a>.

- **Automatic bug filing**: A build system can use result matching to identify new results, and then automatically file bugs for them.<sup><a href="#note-1">1</a></sup>

SARIF defines a wide variety of objects and properties, most of which are optional.
Depending on what the log file will be used for, various subsets of those properties are important.

## The playbook

The purpose of the playbook is:

- To enable SARIF producers to create log files that are fit for a specific purpose.
- To enable SARIF consumers to confirm that the log files they receive are fit for purpose before attempting to use them for that purpose.

For example, a build system might need to produce log files that are suitable for automatic bug filing, and an automatic bug filing system would need to ensure that the log files it receives are suitable for automatic bug filing before attempting to file bugs from them.

The playbook consists of the following steps.

### <a id="step-1">Step 1</a>: Define and document a standard for the usage scenario

Specify the SARIF objects and properties necessary to support the scenario.
    
If necessary, specify any constraints on the required properties beyond those defined by SARIF itself. For example, it might be necessary for a URI-valued property to contain a relative reference rather than an absolute URI.
   
Explain the value of each object, property, and constraint in the scenario.

### <a id="step-2">Step 2</a>: Identify and/or implement analysis rules to verify fitness for purpose

The [SARIF Multitool](Multitool.md) offers a `validate` command that runs a set of analysis rules on a SARIF log file, and produces another log file with the results:

    sarif validate MyFile.sarif --output MyFile-validation.sarif

The complete set of existing analysis rules is available in the [src/Sarif.Multitool/rules](https://github.com/microsoft/sarif-sdk/tree/master/src/Sarif.Multitool/Rules) directory in the [microsoft/sarif-sdk](https://github.com/microsoft/sarif-sdk) repo.

For each element of the fitness standard identified in <a href="#step-1">Step 1</a>, identify a validation rule that enforces it. If no such rule exists, then:

- If the rule is generally useful, feel free to contribute a new rule to the sarif-sdk repo.
- If the rule is specific to a single, proprietary system, implement the rule in a plug-in rules assembly.<sup><a href="#note-2">2</a></sup>

If you are contributing a new rule, consider whether the rule is useful in all scenarios (in which case it should be enabled by default) or is useful only in this particular scenario (in which case it should be disabled by default).

In some cases, it might not be practical to implement an analysis rule for an element of the standard.
For example, the standard might impose certain quality requirements on user-facing messages that are difficult to validate automatically.

Document the set of analysis rules that validates fitness for purpose. Call out any fitness criteria for which a rule cannot be written.

### Step 3: Provide fixes for issues identified by the analysis rules

Provide a programmatic mechanism for fixing (as far as possible) the issues identified by the analysis rules listed in <a href="#step-2">Step 2</a>.

The SARIF Multitool offers a `rewrite` command that can perform a variety of operations on a SARIF log file, such as populating optional SARIF properties (we refer to this operation as <a href="Glossary.md#enrichment">_enrichment_</a>). For example, you can embed the contents of the analyzed files into the SARIF log file:

    sarif rewrite MyFile.sarif --insert TextFiles --output MyFile-enriched.sarif

Identify any existing `rewrite` options that help to make a SARIF log file fit for purpose. These might include one or more of the arguments to the `--insert` option, or other existing options to the `rewrite` command.

If there is no command line option that performs the necessary operations, consider contributing a configurable API to the SARIF SDK and integrating it into the `rewrite` command -- either as an additional argument to the `--insert` option, or as a separate command line option.<sup><a href="#note-3">3</a></sup>

Document the complete set of `rewrite` options that fix the issues identified by the analysis rules. If any other programmatic or manual steps are necessary to make the log file fit for purpose, document those as well.

### Step 4: Provide a configuration file that enables just the relevant rules.

Provide an XML configuration file that

- Enables the analysis rules identified in <a href="#step-1">Step 1</a>.
- Configures the rules, if necessary, with appropriate parameter values.
- Disables all other rules with rule ids `SARIF2000` and above.<sup><a href="#note-4">4</a></sup>

If appropriate, contribute the configuration file to the [policies](https://github.com/microsoft/sarif-sdk/tree/master/policies) directory of the [microsoft/sarif-sdk](https://github.com/microsoft/sarif-sdk) repo.

You can use [policies/gitub-dsp.config.xml](https://github.com/microsoft/sarif-sdk/tree/master/policies/github-dsp.config.xml) as a model.<sup><a href="#note-5">5</a></sup>

## Summary

That's it. SARIF producers can use the `sarif rewrite` command defined in <a href="#step-3">Step 3</a> to produce SARIF log files that are fit for purpose. SARIF consumers can use the `sarif validate` command with the configuration file defined in <a href="#step-4">Step 4</a> verify that the files it receives are fit for purpose before it processes them further. During development, SARIF producers can do the same to ensure that their `rewrite` operation works properly.

## Notes

<a id="note-1">1.</a> The system doesn't need to file a separate bug for each result. It might file one bug per source file, or even one bug per tool run.

<a id="note-2">2.</a> At the time of writing, plug-in rule assemblies are not supported. Issue [microsoft/sarif-sdk#1868](https://github.com/microsoft/sarif-sdk/issues/1868), "Allow validation command to accept plug-ins with additional validation rules", tracks this work. Feel free to raise a PR to address it!

<a id="note-3">3.</a> At the time of writing, we consider the `rewrite` command the appropriate place to integrate operations that make SARIF log files fit for purpose. It is possible that a future version of the SARIF Multitool will provide a different way of accessing this functionality.

<a id="note-4">4.</a> Analysis rules in the range `SARIF1001-1999` represent absolute requirements of the SARIF standard. They are enabled by default and should never be disabled.

<a id="note-5">5.</a> At the time of writing, this config file does not follow the guidance of disabling non-relevant 2000-level rules. Issue [microsoft/sarif-sdk#2039](https://github.com/microsoft/sarif-sdk/issues/2039), "Disable non-relevant 2000-level rules in DSP config files", tracks this work. Feel free to raise a PR to address it!

[Table of contents](../README.md#contents)