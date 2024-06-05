# Build Type: GitHub Actions Workflow

This is a GitHub-maintained [SLSA Provenance](https://slsa.dev/provenance/v1)
`buildType` that describes the execution of a GitHub Actions workflow.

## Description

```jsonc
"buildType": "https://actions.github.io/buildtypes/workflow/v1"
```

This `buildType` describes the execution of a top-level [GitHub Actions]
workflow that builds a software artifact.

Only the following [event types] are supported:

| Supported event type  | Event description                          |
| --------------------- | ------------------------------------------ |
| [`create`]            | Creation of a git tag or branch.           |
| [`release`]           | Creation or update of a GitHub release.    |
| [`push`]              | Creation or update of a git tag or branch. |
| [`workflow_dispatch`] | Manual trigger of a workflow.              |

[`create`]:
  https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#create
[`release`]:
  https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#release
[`push`]:
  https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#push
[`workflow_dispatch`]:
  https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#workflow_dispatch

This build type MUST NOT be used for any other event type unless this
specification is first updated. Other event types are irrelevant for software
builds (such as `issues`) or have complex semantics that may be error prone or
require further analysis (such as `pull_request` or `repository_dispatch`). To
add support for another event type, please open a [GitHub Issue].

Note: Consumers SHOULD reject unrecognized external parameters, so new event
types can be added without a major version bump as long as they do not change
the semantics of existing external parameters.

[GitHub Issue]: https://github.com/actions/buildtypes/issues
[GitHub Actions]: https://docs.github.com/en/actions
[event types]:
  https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows

## Build Definition

### External parameters

[External parameters]: #external-parameters

All external parameters are REQUIRED unless empty.

<table>
<tr><th>Parameter<th>Type<th>Description

<tr id="workflow"><td><code>workflow</code><td>object<td>

The workflow that was run.

<tr id="workflow.ref"><td><code>workflow.ref</code><td>string<td>

A git reference to the commit containing the workflow, as either a git ref
(starting with `refs/`). This is the value passed in via the event.

Can be computed from the [github context] using `github.ref`. Note that
`github.ref` is not guaranteed to be available for all event types but should be
present for all currently supported event types.

<tr id="workflow.repository"><td><code>workflow.repository</code><td>string<td>

HTTPS URI of the git repository, with `https://` protocol and without `.git`
suffix.

Can be computed from the [github context] using
`github.server_url + "/" + github.repository`.

<tr id="workflow.path"><td><code>workflow.path</code><td>string<td>

The path to the workflow YAML file within the commit.

Can be computed from the [github context] using `github.workflow_ref`, removing
the prefix `github.repository + "/"` and the suffix `"@" + github.ref`. Take
care to consider that the path and/or ref MAY contain `@` symbols.

</table>

[github context]:
  https://docs.github.com/en/actions/learn-github-actions/contexts#github-context
[release body parameters]:
  https://docs.github.com/en/rest/releases/releases?apiVersion=2022-11-28#create-a-release--parameters
[variables]: https://docs.github.com/en/actions/learn-github-actions/variables
[vars context]:
  https://docs.github.com/en/actions/learn-github-actions/contexts#vars-context

### Internal parameters

All internal parameters are OPTIONAL.

| Parameter | Type   | Description                                                                                                                                                               |
| --------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `github`  | object | A subset of the [github context] as described below. Only includes parameters that are likely to have an effect on the build and that are not already captured elsewhere. |

The `github` object SHOULD contains the following elements:

| GitHub Context Parameter     | Type   | Description                                                                                             |
| ---------------------------- | ------ | ------------------------------------------------------------------------------------------------------- |
| `github.event_name`          | string | The name of the event that triggered the workflow run.                                                  |
| `github.repository_id`       | string | The numeric ID corresponding to `externalParameters.workflow.repository`.                               |
| `github.repository_owner_id` | string | The numeric ID of the user or organization that owns `externalParameters.workflow.repository`.          |
| `github.runner_environment`  | string | The type of runner on which the the workflow was executed. One of the `github-hosted` or `self-hosted`. |

Numeric IDs are used here to provide stable identifiers across account and
repository renames and to detect when an old name is reused for a new entity.

### Resolved dependencies

The `resolvedDependencies` SHOULD contain an entry identifying the resolved git
commit ID corresponding to `externalParameters.workflow`. The dependency's `uri`
MUST be in [SPDX Download Location] format:

```
"git+" + github.server_url + "/" + github.repository + "@" + github.ref
```

See [Examples](#examples).

[SPDX Download Location]:
  https://spdx.github.io/spdx-spec/v2.3/package-information/#77-package-download-location-field

## Run details

### Builder

The `builder.id` MUST represent the entity that generated the provenance, as per
the [SLSA Provenance](https://slsa.dev/provenance/v1#builder.id) documentation.
In practice, this is the workflow responsible for assembling/signing the
provenance. When the provenance is generated within a [Reusable Workflow] that
workflow will be used as the `builder.id`. Otherwise, it will refer to the
top-level workflow:

```
github.server_url + "/" + github.job_workflow_ref
```

[Reusable Workflow]:
  https://docs.github.com/en/actions/using-workflows/reusing-workflows

### Metadata

The `invocationId` SHOULD be set to:

```
github.server_url + "/" + github.repository "/actions/runs/" + github.run_id + "/attempts/" + github.run_attempt
```

## Examples

See [example.json](example.json).

## Version history

### v1

Initial version
