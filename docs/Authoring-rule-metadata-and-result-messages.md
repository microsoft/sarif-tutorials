[Table of contents](../README.md#contents)

# Appendix A: Authoring rule metadata and result messages

This Appendix explains how to author the metadata that describes a tool's analysis rules.
Perhaps the most important component of this metadata is a result message that is informative and actionable.
Rule metadata must provide enough information to help users understand and fix the issues reported by the tool,
and to enable automated systems to perform functions such as bug filing.

We will describe the most important components of rule metadata, and explain how SARIF represents them.
In the SARIF spec, almost every one of these components is optional;
only the ["stable, opaque identifier"](#rule-id) is required. But for a rule to be useful, it should include all these components
except for those that we explicitly call out below as "optional".

## <a id="rule-id"></a>Stable, opaque identifier

Every rule must have a stable, opaque identifier, stored in the rule's `id` property.
"Stable" means that once `id` is established for a given rule, it must never change.
"Opaque" means that it's not intended to be human-readable or parseable into smaller components.
It typically consists of an abbreviation for the tool name followed by a numeric code.

Example: `BA2006`

These identifiers must never be reused.
If you decide `BA2006` was a terrible rule and you remove it, you can’t name the next rule you create `BA2006`.
Just leave a gap in the numbering.

## <a id="rule-name"></a>Human-readable identifier

A rule may also have a human-readable identifier, a concise but understandable identifier for the rule,
stored in the rule's `name` property.
It should be as short as possible while maintaining readability:
no more than 5 or 6 words, ideally shorter than 30 characters,
and only longer than 40 characters when absolutely necessary.

It generally takes one of two forms:

- A description of a negative condition flagged in code, _e.g._, `IdentifierIsMisspelled`.

- A general guideline that was not followed, _e.g._, `TypeNamesArePascalCased`.

Example: `BuildWithSecureTools`

## <a id="rule-short-description"></a>Short description

The short description, stored in the rule's `shortDescription` property, is a brief summary
that captures the essential issue or guidance embodied in the rule.
It should consist of a single sentence.
It is intended to be displayed in UI contexts where space is limited.

Example: `Compile your code with the latest tool set.`

## <a id="rule-full-description"></a>Full description

The full description is a more comprehensive description of the rule,
stored in the rule's `fullDescription` property,
that includes a broader technical and runtime context for the problem or principle it relates to.
It should include information such as:

- An explanation of why the condition flagged by the rule is a problem.

- An exhaustive set of suggestions for mitigating the problem.

- Circumstances in which the issue can be ignored, and circumstances in which it should never be ignored or suppressed.

Example:

    Application code should be compiled with the most up-to-date tool sets possible to take advantage of the most current compile-time security features. Among other things, these features provide address space layout randomization, help prevent arbitrary code execution, and enable code generation that can help prevent speculative execution side-channel attacks.

## <a id="rule-level"></a>Level

"Level" describes the severity of the problem. It is typically one of `"note"`, `"warning"`, or `"error"`.
SARIF actually stores this information in a sub-object of the rule, so you will find it at `defaultConfiguration.level`

## <a id="rule-message-strings"></a>Message strings

Message strings provide information about each specific instance of a rule violation.
A rule can have more than one message string for two reasons:

- Some rules report two or more distinct error conditions.
- Some tools report "pass" results, that is,
    they provide an explicit signal that a rule was applied to a target, and the target passed the test.
    Such a rule will have both a "pass" message and a "fail" message.

Message strings can contain substitution sequences (`{0}`) which are usually surrounded by single quotes.
You can omit the quotes when it makes sense, for example:

- When the message string includes a dynamic message after a colon, for example,
    `"An exception was raised by the XML parser: {0}"`.
- When the value is numeric, for example,
    `"The line length of %d exceeds the maximum length of %d."`

Message strings must be grammatically correct, correctly spelled,
and consist of one or more complete sentences terminated by periods.
All message strings produced by a given tool should use consistent terminology,
for example, referring to an HTML "tag" or an "element" but not both.

A message string must contain enough information to enable a user to resolve the problem. This includes:

- Sufficient details to locate the analysis target that was flagged as problematic.
    In addition to the file and line information that SARIF provides alongside the message,
    it is helpful to include other identifying characteristics such as a relevant HTML element name or a function name.

- A precise description of why the analysis target was flagged.

- A summary of why the pattern is poor practice (particularly in contexts such as security or accessibility
    where driving considerations might not be readily apparent).

- Guidance for resolving the problem. This guidance should include all reasonable possibilities
    (including describing circumstances in which ignoring the problem altogether might be appropriate).

- Special considerations such as:

    - Noting when a violation should never be ignored or suppressed.
    - Noting when a violation could cause downstream tool noise/false positives.
    - Noting when a rule can be configured in some way to refine/alter analysis.

Plain text messages should be expressed in a single paragraph, without line breaks or formatting
elements such as HTML tags or Markdown markup.
Markdown messages, on the other hand, may make full use of Markdown's formatting capabilities
(for example, line breaks, lists, code fragments, and italic or bold text).
Because of this, it might make sense to present the information in the Markdown version of a message
in a different order.

In addition to providing the required information and conforming to the stylistic guidelines
(complete sentences, correct grammar and spelling, etc.),
message strings should follow certain "presentational" guidelines:

- The first sentence should provide a concise summary of the issue.
This is important because it enables the reader to quickly understand the problem
and because some viewers might truncate the message to one sentence in contexts where screen real estate is limited.

- Design the first sentence for people who are familiar with this class of issue,
but provide additional explanatory detail later for people who are not.
This optimizes the "read many" case.
The first time a user encounters the issue, they can study the entire message to understand it.
After that, a quick glance at the first sentence tells them all they need to know.

- If there is a common term for this type of issue
    (for example, "resource inclusion attack" or "cross-site scripting"),
    then mention it in the first sentence.
    This helps the user look up the term in a web search.

- Speaking of searches, don't hesitate to include hyperlinks to reliable sources
    to aid the user's search for information.
    SARIF allows even plain text messages to include hyperlinks using a subset of the Markdown syntax:

        This page might be vulnerable to a [forced browsing attack](https://owasp.org/www-community/attacks/Forced_browsing).

    which a conformant SARIF viewer will render as
    "This page might be vulnerable to a
    [forced browsing attack](https://owasp.org/www-community/attacks/Forced_browsing)."

- Don’t overstate the certainty of the result.
    If the message states a problem with certainty when the tool is not in fact certain that a problem exists,
    users might lose confidence in the tool.
    In these cases, use softer language.
    For example, rather than stating that a variable "can be controlled by an attacker,"
    you might write "may be controlled by an attacker."

The following examples conforms to these informational, stylistic, and presentational guidelines:

Example 1

Plain text:

    The page at '{0}' might be vulnerable to a resource inclusion attack. It uses script to construct the path to a file which it then downloads and executes. Some portion of the path may be controlled by an attacker (for example, because it contains user input), which would allow arbitrary code execution. Ensure that all user-settable portions of all resource paths are sanitized.

Markdown:

    The page at '{0}' might be vulnerable to a resource inclusion attack. It uses script to construct the path to a file which it then downloads and executes. Some portion of the path may be controlled by an attacker (for example, because it contains user input), which would allow arbitrary code execution.
    
    ## Recommendation
    
    Ensure that all user-settable portions of all resource paths are sanitized.

Example 2:

Plain text:

    '{0}' was compiled with one or more modules which were not built using minimum required tool versions. More recent tool versions contain mitigations that make it more difficult for an attacker to exploit vulnerabilities in programs they produce. To resolve this issue, compile and/or link your binary with compiler version '{2}' or later. If you are servicing a product where the tool chain cannot be modified (e.g., producing a hotfix for an already shipped version) ignore this warning. Modules built outside of policy: '{2}'.

Markdown:

    '{0}' was compiled with one or more modules which were not built using minimum required tool versions. More recent tool versions contain mitigations that make it more difficult for an attacker to exploit vulnerabilities in programs they produce.

    The following modules were build outside of policy: {2}
    
    # Recommendation
    
    - Compile and/or link your binary with compiler version '{2}' or later.
    - If you are servicing a product where the tool chain cannot be modified (_e.g._, producing a hotfix for an already shipped version) ignore this warning.

## <a id="rule-tags"></a>Categories ("tags")

You can optionally specify a one or more tags that categorize the rule.

Examples: `"security"`, `"build-options"`

## <a id="rule-help-uri"></a>Help URI

If it exists, you should specify in the rule's `helpUri` property the URI of a web page
that provides more information about the rule and possibly links to additional resources.

## SARIF representation

Here is what the example rule we have been following looks like in SARIF.

```json
{
  "version": "2.1.0",
  "runs": [
    {
      "tool": {
        "driver": {
          "name": "BinSkim",
          "version": "1.6.0",
          "rules": [
            {
              "id": "BA2006",
              "name": "BuildWithSecureTools",
              "shortDescription": {
                "text": "Compile your code with the latest tool set."
              },
              "fullDescription": {
                "text": "Application code should be compiled with..."
              },
              "defaultConfiguration": {
                "level": "warning"
              },
              "messageStrings": {
                "Error_ModuleVersion": {
                  "text": "'{0}' was compiled with one or more modules..."
                },
                "Error_CompilerVersion": {
                  "text": "'{0}' was built with {1} compiler version {2}."
                },
                "Pass": {
                  "text": "All modules of '{0}' satisfy configured policy. "
                }
              },
              "helpUri": "https://binskim/rules/BA2006.html",
              "properties": {
                "tags": [
                  "security",
                  "build-options"
                ]
              }
            },
            ...
          ]
        }
      },
      "results": [ ... ]
    }
  ]
}
```

[Table of contents](../README.md#contents)
