# Utilities

SCRUB provides several different utilities that can be used generically in many different scenarios.


## Translation Utility

`scrub.tools.parsers.translate_results` is a utility tool for translating analysis results to and from SARIF/SCRUB. The analysis results are parsed into a temporary in-memory representation of the results which is identical regardless of the input file type. This in-memory representation is then used to translate warnings in any direction to output either SCRUB or SARIF results.

This process can also be used to translate between SARIF versions, of which SCRUB currently supports sarifv2.1.0 and sarifv2.0.0. Additionally, the translator supports result merging for SARIF files, which can combine warnings from multiple analysis tool executions into a single results file. Below are the relevant modules for these of the translation tool.


### Functionality and Usage

There are four core pieces of functionality that are contained within the SARIF translation tool

1. Translation from SCRUB to SARIF output format style  

    `scrub.utils.translate_results.perform_translation('<tool>.scrub', '<tool>.sarif', '<source root>', 'sarifv2.1.0')`

2. Translate from SARIF to SCRUB output format style

    `scrub.utils.translate_results.perform_translation('<tool>.sarif', '<tool>.scrub', '<source root>', 'scrub')`

3. Translate from one version of SARIF to another version of SARIF

    `scrub.utils.translate_results.perform_translation('<tool>.sarif', '<tool>.sarif', '<source root>', 'sarifv2.1.0')`

4. Merge several SARIF files into a single SARIF output file

    `scrub.utils.translate_results.run_sarif_merge('<input 1>.sarif', '<input 2>.sarif', '<output>.sarif', 'sarifv2.1.0')`


## Diff Utility

`utils.diff_results` is an entirely standalone tool that can be used to diff two sets of SCRUB results. This allows teams to establish a baseline set of static analysis findings to diff against so new findings can be easily identified. This utility can be run from either as a Python module or directly from the command line:

    python3 -m scrub.utils.diff_results <Baseline Source Root> <Baseline Results Root> <Comparison Source Root> <Comparison Results Root>

    scrub diff --baseline-source <Baseline Source Root> --baseline-scrub <Baseline Results Root> --comparison-source <Comparison Source Root> --comparison_scrub <Comparison Results Root>

Four input values are required for execution:

* *Baseline Source Root*: Absolute path to the root directory of the baseline source code
* *Baseline Results Root*: Absolute path to the root directory of the baseline SCRUB results
* *Comparison Source Root*: Absolute path to the root directory of the comparison source code
* *Comparison Results Root*: Absolute path to the root directory of the comparison SCRUB results

One use case for the diff utility would be to insert it into execution as a `CUSTOM_FILTER_CMD` to isolate new results when compared to a baseline set of results. The configuration file would look something like this::

    [Filtering Variables]
    ENABLE_EXT_WARNINGS: False
    ENABLE_MICRO_FILTER: True
    CUSTOM_FILTER_CMD: scrub diff --baseline-source <Baseline Source Root> --baseline-scrub <Baseline Results Root> --comparison-source <Comparison Source Root> --comparison_scrub <Comparison Results Root>
    ANALYSIS_FILTERS:
    QUERY_FILTERS:

In this case the Comparison Source Root and Comparison SCRUB Results should be `SOURCE_DIR` and `SOURCE_DIR/.scrub` respectively

The diff utility will identify two different types of matches between the results sets. Any finding that fits into one of the categories below will be printed out to a new SCRUB formatted file located at `comparison_scrub/<tool>_diff.scrub`. Any finding that does not fit into one of these categories should be consider a new finding.

* Exact match: These are results in the comparison set that are generated by the same tool, using the same query, and appear on the same line of the same source file
* Probable match: These are results in the comparison set that are generate by the same tool, using the same query, but appear on a different line within the source file