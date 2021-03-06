# Laminas Continuous Integration GitHub Action

This repository represents a GitHub action that runs a single job in the [laminas/laminas-continuous-integration-action](https://github.com/laminas/laminas-continuous-integration-action).

A job is presented as an argument to the container, and will be a JSON string representing the job to run.

> **NOTE** that it is a JSON string representation, and not an actual JSON object.

The JSON string should represent an object with the following information:

```json
{
  "php": "string PHP minor version to run against",
  "extensions": [
    "extension names to install; names are from the ondrej PHP repository, minus the php{VERSION}- prefix",
  ],
  "ini": [
    "php.ini directives, one per element; e.g. 'memory_limit=-1'",
  ],
  "dependencies": "dependencies to test against; one of lowest, locked, latest",
  "command": "command to run to perform the check",
}
```

The PHP version and command are required; all other elements are optional.

It will then execute the job, and the exit status will determine job failure or success.

Generally speaking, you will use this in combination with the [laminas/laminas-ci-matrix-action](https://github.com/laminas/laminas-ci-matrix-action), which will build a matrix of jobs for you based on configuration files already present in your package.

## Usage

```yaml
jobs:
  matrix:
    name: Generate job matrix
    runs-on: ubuntu-latest
    outputs:
      matrix: ${{ steps.matrix.outputs.matrix }}
    steps:
      - name: Gather CI configuration
        id: matrix
        uses: laminas/laminas-ci-matrix-action@v1

  qa:
    name: QA Checks
    needs: [matrix]
    runs-on: ${{ matrix.operatingSystem }}
    strategy:
      fail-fast: false
      matrix: ${{ fromJSON(needs.matrix.outputs.matrix) }}
    steps:
      - name: ${{ matrix.name }}
        uses: laminas/laminas-continuous-integration-action@v1
        with:
          job: ${{ matrix.job }}
```

> ### actions/checkout not required
>
> The laminas/laminas-continuous-integration-container will perform a checkout of the repository at the requested reference as part of its work, and therefore does not require the actions/checkout action as a preceding step.

### Using locally

Because the action requires input to work, you will need to pull the [laminas/laminas-continuous-integration-container] locally and run your job via the container directly.
Please see the [container documentation for details](https://github.com/laminas/laminas-continuous-integration-container#using-the-container-locally-to-run-tests).
