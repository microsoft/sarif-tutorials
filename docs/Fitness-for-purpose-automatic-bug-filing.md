[Table of contents](../README.md#contents)

# Appendix: Fitness for purpose: Automatic bug filing

## Introduction

This Appendix describes how to produce SARIF log files that are <a href="Glossary.md#fit-for-purpose">_fit for purpose_</a> for automatic bug filing. It is an example of the "playbook" for producing fit-for-purpose SARIF log files, described in the Appendix [Fitness for purpose: Overview](Fitness-for-purpose-overview.md).

The standard described here works well for a particular bug filing system used internally at Microsoft.
You might be able to use it without change, or you might have to adapt it to your company's engineering system.
In that case, do read the [Overview](Fitness-for-purpose-overview.md) to understand the purpose of each component of the playbook given here.

## The standard

For automatic bug filing to be effective, the engineering system must compare the SARIF results from the current build to those from a previous build, to identify those results that are new in the current build. This process is known as <a href="Glossary.md#result-matching">_result matching_</a>. Some of the requirements of the bug filing fitness-for-purpose standard help to ensure that result matching is accurate, reducing the number of duplicate bugs that are filed (and ensuring that bugs _are_ filed for truly new results).

In the following list of requirements, "should" means that the validator should produce a `"warning"`-level message, and "must" means that the validator should produce an `"error"`-level message. In any case where this is not the default behavior for the rule, the configuration file (see below) can adjust the rule's level.

- Result `message` objects should provide the `id` and `arguments` properties (as opposed to only providing the `text` property). This is important because the result macthing algorithm (implemented in the [SARIF Multitool's](Multitool.md) `match-results-forward` command) includes message id and arguments in its matching criteria.

- The `run` object must provide the `versionControlProvenance` property. The bug filer uses this information (which includes the URL of the repo containing the files that were analyzed) to decide which team should be assigned the filed bugs.

- The `run.tool.driver` object must provide at least one of the `version`, `semanticVersion`, or `dottedQuadFileVersion` properties.

- The `run.tool.driver` object should provide `informationUri`. This helps the developer responsible for fixing the bug by providing a way to learn more about the tool.

- All result location URIs should be expressed as relative references with respect to the repo root. **WHY**

- Each rule should provide `helpUri`. This helps the developer responsible for fixing the bug by providing a way to learn more about the rule violation.

- Each result should provide a way for a user viewing the bug to see the problem in context. This can be done either by including a code snippet (`result.locations[].physicalLocation.{region,contextRegion}.snippet`) or by embedding the entire file... **UNFINISHED**

## The analysis rules

The analysis rules that enforce these standards are:

- `SARIF2002.ProvideMessageArguments`: ensures that `result.message.id` and `.arguments` are present.
- `SARIF2003.ProvideVersionControlProvenance`: ensures that `run.versionControlProvenance` is present.
- `SARIF2005.ProvideToolProperties`: ensures that `run.tool.driver.name`, and at least one of `.version` and `.semanticVersion`, is present.
- `SARIF2007.ExpressPathsRelativeToRepoRoot`: ensures that all result location URIs are expressed as relative references with respect to the repo root.
- `SARIF2012.ProvideHelpUris`: ensures that `rules[].helpUri` is present.
- `SARIF2013.ProvideEmbeddedFileContent OR SARIFnippets`: ensures that ... **UNFINISHED**

## The fixers

## The configuration file

This is a standard validation XML configuration file, which explicitly enables all the analysis rules mentioned above, and sets there levels as indicated in the standard section above.

[Table of contents](../README.md#contents)