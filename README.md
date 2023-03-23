# xonsh/actions

Github CI workflows for the xonsh shell projects.

## Usage example

Example from [xontrib-abbrevs](https://github.com/xonsh/xontrib-abbrevs/blob/5fbd8bfaf7cab2ebda05bb8294e6fc14e65f29e5/.github/workflows/test.yml):

```yaml
name: Testing

on:
  push:
  pull_request:

jobs:
  testing:
    # reuse workflow definitions
    uses: xonsh/actions/.github/workflows/test-pip-xontrib.yml@main
    with:
      cache-dependency-path: pyproject.toml

```
