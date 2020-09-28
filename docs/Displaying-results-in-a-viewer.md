[Table of contents](../README.md#contents)

# Displaying results in a viewer

Many SARIF viewers display a list of the results contained in the log file or files being viewed. Since log files can contain thousands of results, it is important for the viewer to display by default just those results that are most useful, while still giving the user a way to see all available results. Even if a log file contains only a few results, the viewer can still help the user by directing her attention to the most important ones.

Having decided which results to display, most viewers have a way to indicate the relative importance of each result. The viewer should direct the user's attention to the most important of the displayed results.

This document describes how a viewer should decide which results to show by default,
and how it should convey their relative importance. The viewer should provide some UI gesture to allow the user to see results that are not shown by default.

## Determining which results to display

Four properties of the SARIF `result` object interact to determine which results to show by default:

- `suppressions`
- `baselineState`
- `kind` and `level`, combined as described in the section <a href="#importance">Determining importance</a> to produce an `importance` value.

Proceed through the algorithm below. If at any point the algorithm states that the result should not be shown by default, there is no need to proceed any further (you will see the word "STOP"). If at any point the algorithm states that the result _should_ be shown by default, it is possible that a subsequent decision point in the algorithm might still decide not to show the result. Proceed until you come to the end of the algorithm or you encounter a STOP.

1. `result.suppressions`: If the result is suppressed as described in the section <a href="#suppression-status">Determining suppression status</a>, it should not be shown by default. STOP.

2. `result.baselineState`:

    - If the baseline state is `"absent"` (meaning that the result appeared in the previous run but no longer appears in the current run) then it should not be shown by default. STOP.

    - If the baseline state is `"new"` (meaning that the result did not appear in the previous run but has appeared for the first time (or perhaps reappeared) in this run) then it should be shown by default.

    - If the baseline state is `"unchanged"` (meaning that the result has not changed since the previous run) or `"updated"` (meaning that some aspects of the result have changed since the previous run, but it is nonetheless still identifiably the same result), then whether or not to show the result by default depends on the usage scenario.

        If the viewer is operating in a continuous integration (CI) workflow, where the developer wants to see whether he has introduced any problems in his most recent commits, then `"unchanged"` and `"updated"` results should not be shown by default. STOP.

        Otherwise, `"unchanged"` and `"updated"` results should be shown by default.

    - If `result.baselineState` is absent (meaning that this run has not been "baselined" -- compared to a previous run -- and therefore there is no way for the viewer to know if the result is new or pre-existing) then the result should be shown by default.

3. `importance`:

    - If `importance` is `"note"`, the result should not be shown by default. STOP.
    - If `importance` is `"warning"` or `"error"`, the result should be shown by default.

## <a id="suppression-status"></a>Determining suppression status

The `result.suppressions` property is an array each of whose elements represents a request to suppress the result. Each array element has an optional `"status"` property that describes the review status of the suppression request: `"accepted"`, `"underReview"`, or `"rejected"`.<sup><a href="#note-1">1</a></sup>

If the status of _any_ of the suppressions is `"underReview"` or `"rejected"`, then the result should _not_ be considered suppressed. Otherwise, the result _should_ be considered suppressed.

## <a id="importance"></a>Determining importance

The SARIF `result` object's `level` and `kind` properties together determine the result's  overall importance. For clarity, we refer to the result of this importance calculation as `importance`, set in code font, even though the `result` object does not have an "importance" property. `importance` is always one of `"error"`, `"warning"`, or `"note"`.

In general, results whose `importance` is `"error"` are blocking and must be fixed, results whose `importance` is `"warning"` are non-blocking but should still be fixed, and results whose `importance` is `"note"` represent either optional opportunities to improve code quality, or purely informational messages.

By the SARIF spec, only certain combinations of `kind` and `level` are valid. If `level` is `"error"`, `"warning"`, or `"note"`, then the result describes a failure of some kind, and therefore `"kind"` must be `"fail"`. Conversely, if `"kind"` is anything other than `"fail"` (for example `"pass"` or `"open"`), then the result does _not_ describe a failure, so `"level"` must be `"none"`.<sup><a href="#note-2">2</a></sup>

With that background, `importance` is calculated as follows:

If `level` is `"error"`, `"warning"`, or `"none"`, then `importance` is equal to `"level"`.

If `level` is `"none"`, `importance` depends `kind` as follows:

| `kind` | Meaning | `importance` |
| --- | --- | --- |
| `"informational"` | The result represents a purely informational observation about the code (for example, "This file was compiled with C++ Compiler Version 7.1.2") | `"note"` |
| `"notApplicable"` | The analysis rule was not applicable to this analysis target. | `"note"` |
| `"open"` | The analysis tool did not have enough information to determine whether there was a problem.<sup><a href="#note-3">3</a></sup> | `"warning"` |
| `"pass"` | The analysis rule was evaluated, and no problem was found. | `"note"`<sup><a href="#note-4">4</a></sup> |
| `"review"` | The result requires human review | `"warning"` |

## Summary

To ensure a uniform experience across the SARIF ecosystem, it is important that viewers implement a standard algorithm for determining which results to display by default. We recommend the algorithm described here.

In addition, viewers should provide UI gestures to allow users to see results that are not displayed by default.

## Notes

<a id="note-1"></a>1. See [§3.35](https://docs.oasis-open.org/sarif/sarif/v2.1.0/os/sarif-v2.1.0-os.html#_Toc34317733) and [§3.35.3](https://docs.oasis-open.org/sarif/sarif/v2.1.0/os/sarif-v2.1.0-os.html#_Toc34317736) in the SARIF spec for details.

<a id="note-2"></a>2. See [§3.27.9](https://docs.oasis-open.org/sarif/sarif/v2.1.0/os/sarif-v2.1.0-os.html#_Toc34317647) and [§3.27.10](https://docs.oasis-open.org/sarif/sarif/v2.1.0/os/sarif-v2.1.0-os.html#_Toc34317648) in the SARIF spec for details.

<a id="note-3"></a>3. This value is used by program correctness provers. If a prover doesn't know (for example), if a particular function can throw an exception, it might not be able to decide whether a code path is safe.

<a id="note-4"></a>4. If possible, a viewer should represent a `"pass"` result with an icon distinct from the typical "blue information" icon used for other `"note"`s, for example, a green check mark.

[Table of contents](../README.md#contents)