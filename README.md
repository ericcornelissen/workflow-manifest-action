# Trying (and Failing) to Create a GitHub Actions Workflow Manifest

In GitHub Actions it is possible to create a CI/CD workflow by creating a `.yml`
(or `.yaml`) file in the `.github/workflows` directory. These files will be
picked up by GitHub and executed depending on the specified trigger. Similarly,
creating a reusable GitHub Action just requires putting an `action.yml` (or
`action.yaml`) manifest file somewhere in a repository. Any directory in a
repository with that file can be used as an Action in workflows.

So, what happens if there is a file named `.github/workflows/action.yml`? Can
this file be used both as a workflow and a manifest? tl;dr: No.

A valid workflow file requires a couple of things, an `on` key specifying the
workflow triggers and a `jobs` key specifying the jobs that need to be run as
part of the workflow. Additionally, we can add a `name` to give the workflow a
human-readable name. For example:

```yaml
# Optional
name: Example

# Required
on: [push]

# Required
jobs:
  example:
    name: Example
    runs-on: ubuntu-latest
    steps:
      - name: Example
        run: echo "Example"
```

A valid manifest file requires only a single key: `runs`. Here too it's possible
to give the manifest a human-readable name with the `name` key.

```yml
# Optional
name: Example

# Required
runs:
  using: composite
  steps:
    - name: Example
      shell: bash
      run: echo "Example"
```

These two files can be merged into the file `.github/workflows/action.yml` which
_should_ be picked up by GitHub as a workflow but also be usable as a manifest.

```yml
name: Example

## Workflow
on: [push]
jobs:
  example:
    name: Example
    runs-on: ubuntu-latest
    steps:
      - name: Example
        run: echo "Example"

## Manifest
runs:
  using: composite
  steps:
    - name: Example
      shell: bash
      run: echo "Example"
```

If we do this and try to use it as a GitHub Action it works - Success! However,
when GitHub tries to run this as a workflow it will complaint (regardless of the
location of `runs:` in the source):

```text
Invalid workflow file: .github/workflows/action.yml#L14
The workflow is not valid. .github/workflows/action.yml (Line: 14, Col: 1): Unexpected value 'runs'
```

So, while manifests are permissive and allow additional keys, the syntax of
workflow files prevent this use because additional keys are not allowed. Is this
intentional? Possibly, but it's interesting to see that manifest is not being
restricted in the same way.

If you have any ideas to get this working, feel free to submit an issue.
