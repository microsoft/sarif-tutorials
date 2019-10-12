# SARIF Tutorials

## Introduction

SARIF, the Static Analysis Results Interchange Format, defines a standard format for the output of static analysis tools.
It is a powerful and sophisticated format suited to the needs of a wide variety of tools.
For this reason -- and because the format is defined in a 220-plus page specification written in formal language!
-- it can be hard to learn SARIF and to figure out what parts of it you need to use.

These tutorials aim to present SARIF in a more approachable way.
We'll start with some background:
Why do we need SARIF? Where did it come from? What can it do?
Then we'll dive into the format, exploring the most basic concepts first, then moving on to more advanced concepts.

The advanced concepts usually apply to only a subset of SARIF producers and consumers,
so you don't to read everything.
Just read the introductory material, then pick and choose the additional topics that interest you.

## Disclaimer

The [SARIF specification](https://docs.oasis-open.org/sarif/sarif/v2.1.0/cs01/)
is a [Committee Specification](https://www.oasis-open.org/news/announcements/static-analysis-results-interchange-format-sarif-v2-1-0-from-the-sarif-tc-is-an-a)
from [OASIS](https://www.oasis-open.org/).
But despite the fact that I'm the co-Editor (with Michael Fanning) and primary wordsmith of the specification,
these tutorials are _not_ an OASIS work product or endorsed by OASIS in any way.
They represent my personal interpretation and explanation of the standard.

## Work in progress

I've just started writing these tutorials. I'll remove this notice when I think there's enough information to be useful.

## Table of contents

- [Introduction](docs/Introduction.md)
  - [What is SARIF?](docs/Introduction.md#what-is-sarif)
  - [About static analysis tools](docs/Introduction.md#tools)
  - [Why SARIF?](docs/Introduction.md#why-sarif)
  - [A simple example](docs/Introduction.md#simple-example)
  - [Don't panic!](docs/Introduction.md#dont-panic)
  - [The plan for these tutorials](docs/Introduction.md#plan)
  - [Resources](docs/Introduction.md#resources)
- The basics
  - Logs, runs, and results
  - Messages
  - Physical and logical locations
  - Artifacts
- Beyond the basics
  - Property bags
  - Invocations
  - Notifications
  - Taxonomies
  - Code flows
  - More about messages
- Advanced topics
  - Tool extensions
  - Handling large files
  - Result matching
- Appendices
  - The history of SARIF

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
see the [LICENSE](LICENSE) file, and grant you a license to any code in the repository under the [MIT License](https://opensource.org/licenses/MIT), see the
[LICENSE-CODE](LICENSE-CODE) file.

Microsoft, Windows, Microsoft Azure and/or other Microsoft products and services referenced in the documentation
may be either trademarks or registered trademarks of Microsoft in the United States and/or other countries.
The licenses for this project do not grant you rights to use any Microsoft names, logos, or trademarks.
Microsoft's general trademark guidelines can be found at <http://go.microsoft.com/fwlink/?LinkID=254653.>

Privacy information can be found at <https://privacy.microsoft.com/en-us/>

Microsoft and any contributors reserve all other rights, whether under their respective copyrights, patents,
or trademarks, whether by implication, estoppel or otherwise.
