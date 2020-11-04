# SARIF Tutorials

## Introduction

SARIF, the Static Analysis Results Interchange Format, defines a standard format for the output of static analysis tools.
It is a powerful and sophisticated format suited to the needs of a wide variety of tools.
For this reason &mdash; and because the format is defined in a 220-plus page specification written in formal language!
&mdash; it can be hard to learn SARIF and to figure out what parts of it you need to use.

These tutorials aim to present SARIF in a more approachable way.
We'll start with some background:
Why do we need SARIF? Where did it come from? What can it do?
Then we'll dive into the format, exploring the most basic concepts first, then moving on to more advanced concepts.

The advanced concepts usually apply to only a subset of SARIF producers and consumers,
so you don't to read everything.
Just read the introductory material, then pick and choose the additional topics that interest you.

## Sample files

You can find the sample files displayed in the tutorials under the `samples` directory.
They are all valid SARIF files unless I say otherwise.

## Links to the specification

At times, the tutorials link to a section of the SARIF specification for more detailed information
or descriptions of advanced scenarios.
These links look like this: [§3.13: sarifLog object](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html#_Toc16012434)
and they point into the
[HTML version](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/sarif-v2.1.0-cs01.html) of the spec.
There are also PDF and .docx versions in the [SARIF 2.1.0 CS01](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/)
(Committee Specification 1) folder on the OASIS web site.

The specification is definitive (and if we want to get really technical, the .docx version of the specification is
definitive, but let's assume there are no bugs in the PDF or HTML converters).
If it seems like something in these tutorials disagrees with the spec,
let me know and I'll either fix the tutorials or make sure that a bug is filed against the spec.

## Disclaimer

The [SARIF specification](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/)
is a [Committee Specification](https://www.oasis-open.org/news/announcements/static-analysis-results-interchange-format-sarif-v2-1-0-from-the-sarif-tc-is-an-a)
from [OASIS](https://www.oasis-open.org/).
But despite the fact that I'm the co-Editor (with Michael Fanning) and primary wordsmith of the specification,
these tutorials are _not_ an OASIS work product or endorsed by OASIS in any way.
They represent my personal interpretation and explanation of the standard.

## Status

These tutorials now contain enough information to give you a solid background in SARIF.
As you will see from the "TODO" entries in the table of contents, there's much more I'd like to write about.
But these are advanced topics that I will address when I have time.

If you'd like an explanation of a SARIF feature that I haven't covered,
please let me know by filing an issue in this repo

## <a id="contents"></a>Table of contents

- [Introduction](docs/1-Introduction.md)
  - [What is SARIF?](docs/1-Introduction.md#what-is-sarif)
  - [About static analysis tools](docs/1-Introduction.md#tools)
  - [Why SARIF?](docs/1-Introduction.md#why-sarif)
  - [A simple example](docs/1-Introduction.md#simple-example)
  - [The plan for these tutorials](docs/1-Introduction.md#plan)
- [The basics](docs/2-Basics.md)
  - [The log file and the object model](docs/2-Basics.md#log-file-and-om)
  - [Logs and runs](docs/2-Basics.md#logs-runs)
  - [Tools: driver and extensions](docs/2-Basics.md#tools)
  - [Property bags](docs/2-Basics.md#property-bags)
  - [Results](docs/2-Basics.md#results)
    - [Message](docs/2-Basics.md#message)
    - [Rule identifier](docs/2-Basics.md#rule-id)
    - [Level](docs/2-Basics.md#level)
    - [Locations](docs/2-Basics.md#locations)
      - [The `locations` array](docs/2-Basics.md#loc-array)
      - [Physical and logical locations](docs/2-Basics.md#phys-log-loc)
  - [Artifacts](docs/2-Basics.md#artifacts)
    - [Defining artifacts](docs/2-Basics.md#defining-artifacts)
    - [Linking results to artifacts](docs/2-Basics.md#linking-artifacts)
  - [Rule metadata](docs/2-Basics.md#rule-metadata)
- [Beyond the basics](docs/3-Beyond-basics.md)
  - [Related locations](docs/3-Beyond-basics.md#related-locations)
  - [More about messages](docs/3-Beyond-basics.md#more-about-messages)
    - [Markdown messages](docs/3-Beyond-basics.md#msg-markdown)
    - [Messages from metadata](docs/3-Beyond-basics.md#msg-metadata)
    - [Messages with arguments](docs/3-Beyond-basics.md#msg-args)
    - [Messages with embedded links](docs/3-Beyond-basics.md#msg-links)
      - [Links in text and Markdown](docs/3-Beyond-basics.md#msg-links-text-markdown)
      - [Links to locations](docs/3-Beyond-basics.md#msg-links-location)
  - [Invocations](docs/3-Beyond-basics.md#invocations)
  - [Notifications](docs/3-Beyond-basics.md#notifications)
    - [Tool execution notification and tool configuration notifications](docs/3-Beyond-basics.md#exec-config-notif)
    - [Notifications _vs._ results](docs/3-Beyond-basics.md#notif-result)
  - [Taxonomies](docs/3-Beyond-basics.md#taxonomies)
  - [Code flows](docs/3-Beyond-basics.md#code-flows)
  - [Automation](docs/3-Beyond-basics.md#automation)
    - [The `id` and `guid` properties](docs/3-Beyond-basics.md#run-id-and-guid)
    - [The `correlationGuid` property](docs/3-Beyond-basics.md#run-correlationGuid)
- Advanced topics
  - Tool extensions (TODO)
  - Translations (TODO)
  - Handling large files (TODO)
  - Result matching (TODO)
  - The `sarif:` URI scheme (TODO)
- Appendices
  - [Fitness for purpose: An overview](docs/Fitness-for-purpose-overview.md)
  - [Fitness for purpose: Automatic bug filing](docs/Fitness-for-purpose-automatic-bug-filing.md)
  - [Authoring rule metadata and result messages](docs/Authoring-rule-metadata-and-result-messages.md)
  - [The SARIF Multitool](docs/Multitool.md)
  - The history of SARIF (TODO)
  - [Glossary](docs/Glossary.md)
  - [Resources](docs/Resources.md)

## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit <https://cla.opensource.microsoft.com.>

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Legal Notices

Microsoft and any contributors grant you a license to the Microsoft documentation and other content
in this repository under the [Creative Commons Attribution 4.0 International Public License](https://creativecommons.org/licenses/by/4.0/legalcode),
see the [LICENSE](LICENSE) file, and grant you a license to any code in the repository under the
[MIT License](https://opensource.org/licenses/MIT), see the
[LICENSE-CODE](LICENSE-CODE) file.

Microsoft, Windows, Microsoft Azure and/or other Microsoft products and services referenced in the documentation
may be either trademarks or registered trademarks of Microsoft in the United States and/or other countries.
The licenses for this project do not grant you rights to use any Microsoft names, logos, or trademarks.
Microsoft's general trademark guidelines can be found at <http://go.microsoft.com/fwlink/?LinkID=254653.>

Privacy information can be found at <https://privacy.microsoft.com/en-us/>

Microsoft and any contributors reserve all other rights, whether under their respective copyrights, patents,
or trademarks, whether by implication, estoppel or otherwise.
