# Roadmap

The Serverless Workflow Roadmap.

_Note: Items in tables for each milestone do not imply an order of implementation._

_Note: Milestone entries include the most notable updates only. For list of all commits see [link](https://github.com/cncf/wg-serverless/commits/master)_

_Status description:_

| Completed | In Progress | In Planning | On Hold |
| :--: | :--: |  :--: | :--: |
| ✔ | ✏️ | 🚩 | ❗️|

## v0.1 (Released April 1 2020)

| Status | Description | Comments |
| :--: | --- |  --- |
| ✔ | Establish governance, contributing guidelines and initial stakeholder | [governance doc](https://github.com/cncf/wg-serverless/tree/v0.1/workflow/spec/governance)  |
| ✔ | Define specification goals | [spec doc](https://github.com/cncf/wg-serverless/blob/v0.1/workflow/spec/spec.md) |
| ✔ | Define specification functional scope | [spec doc](https://github.com/cncf/wg-serverless/blob/v0.1/workflow/spec/spec.md) |
| ✔ | Include set of use-cases for Serverless Workflow | [usecases doc](https://github.com/cncf/wg-serverless/blob/v0.1/workflow/spec/usecases.md) |
| ✔ | Include set of examples for Serverless Workflow | [examples doc](https://github.com/cncf/wg-serverless/blob/v0.1/workflow/spec/examples.md) |
| ✔ | Define specification JSON Schema | [spec doc](https://github.com/cncf/wg-serverless/blob/v0.1/workflow/spec/spec.md) |
| ✔ | Add SubFlow state | [spec doc](https://github.com/cncf/wg-serverless/blob/v0.1/workflow/spec/spec.md) |
| ✔ | Add Relay state | [spec doc](https://github.com/cncf/wg-serverless/blob/v0.1/workflow/spec/spec.md) |
| ✔ | Add ForEach state | [spec doc](https://github.com/cncf/wg-serverless/blob/v0.1/workflow/spec/spec.md) |
| ✔ | Update Event state| [spec doc](https://github.com/cncf/wg-serverless/blob/v0.1/workflow/spec/spec.md) |
| ✔ | Define Workflow data input/output | [spec doc](https://github.com/cncf/wg-serverless/blob/v0.1/workflow/spec/spec.md) |
| ✔ | Update state data filtering | [spec doc](https://github.com/cncf/wg-serverless/blob/v0.1/workflow/spec/spec.md) |
| ✔ | Clearly define workflow info passing | [spec doc](https://github.com/cncf/wg-serverless/blob/v0.1/workflow/spec/spec.md) |
| ✔ | Add Workflow error handling | [spec doc](https://github.com/cncf/wg-serverless/blob/v0.1/workflow/spec/spec.md) |
| ✔ | Add reusable function definitions | [spec doc](https://github.com/cncf/wg-serverless/blob/v0.1/workflow/spec/spec.md) |
| ✔ | Add support for YAML definitions | [spec doc](https://github.com/cncf/wg-serverless/blob/v0.1/workflow/spec/spec.md) |
| ✔ | Update workflow end definition | [spec doc](https://github.com/cncf/wg-serverless/blob/v0.1/workflow/spec/spec.md) |
| ✔ | Add Callback state | [spec doc](https://github.com/cncf/wg-serverless/blob/v0.1/workflow/spec/spec.md) |
| ✔ | Add workflow metadata | [spec doc](https://github.com/cncf/wg-serverless/blob/v0.1/workflow/spec/spec.md) |
| ✔️| Update workflow start definition | [spec doc](https://github.com/cncf/wg-serverless/blob/v0.1/workflow/spec/spec.md) |
| ✔️| Prepare github branch and docs for v0.1 | [branch](https://github.com/cncf/wg-serverless/tree/v0.1/workflow/spec) |

## v0.2 (Release data TBD)

| Status | Description | Comments |
| --- | --- |  --- |
| 🚩 | Start discussions on Serverless Workflow Technology Compatibility Kit (TCK) | |
| 🚩 | Decide on state/task/stage/step naming convention | [issue link](https://github.com/cncf/wg-serverless/issues/127) |
| ✏️ | Finish specification primer document | [google doc](https://docs.google.com/document/d/11rD3Azj63G2Si0VpokSpr-1ib3mFRFHSwN6tJb-0LQM/edit#heading=h.paewfy83tetm) |
| ✔ | Update Switch State | [spec doc](spec.md) |
| ✔ | Rename Relay to Inject state | [spec doc](spec.md) |
| ✔️| Update waitForCompletion property of Parallel State | [spec doc](spec.md) |
| ✔️| Add timeout property to actions | [spec doc](spec.md) |
| ✔️| Add examples comparing Argo workflow and spec markups | [examples doc](examples-argo.md) |
| ✔️| Add ability to produce events during state transitions | [spec doc](spec.md) |
| ✔️| Add event-based condition capabilities to Switch State | [spec doc](spec.md) |
| ✔️| Add examples comparing Brigade workflow and spec markups | [examples doc](examples-brigade.md) |
| ✔️| Update produceEvent data property | [spec doc](spec.md) |
| ✔️| Change uppercase property and enum types to lowercase | [spec doc](spec.md) |
| ✔️| Add Parallel State Exception Handling section | [spec doc](spec.md) |
