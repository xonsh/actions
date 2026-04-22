# xonsh/actions

GitHub Actions for the [xonsh](https://xon.sh) shell.

## `xonsh/actions` — setup-xonsh

Composite action that installs xonsh on the runner. Subsequent steps run as xonsh by setting either `shell: xonsh {0}` on a single step 
or `defaults.run.shell: xonsh {0}` on the whole job.

### Usage

#### Per-step shell

Set `shell: xonsh {0}` on the individual step(s) that should run as xonsh — the rest of the job keeps its normal shell:

```yaml
jobs:
  my-job:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: xonsh/actions@main

      - name: Run xonsh
        shell: xonsh {0}
        run: |
          echo "hello from xonsh"
          $PATH.append('/mypath')
          print($PATH)
```

#### Job-level default

Declare `defaults.run.shell: xonsh {0}` once and every subsequent `run:` block in the job executes as xonsh:

```yaml
jobs:
  my-job:
    runs-on: ubuntu-latest
    defaults:
      run:
        shell: xonsh {0}
    steps:
      - uses: actions/checkout@v4
      - uses: xonsh/actions@main

      - run: |
          echo "hello from xonsh"
          $PATH.append('/mypath')
          print($PATH)
```

**Why `{0}`?** GitHub Actions doesn't support a bare `shell: xonsh` — the `{0}` placeholder is required for every non-built-in shell.

### Inputs

| Input | Description | Default |
| --- | --- | --- |
| `xonsh-version` | PyPI version spec (e.g. `0.23.1`, `>=0.23.1`). Empty installs the latest. | *(latest)* |
| `python-version` | If set, runs [`actions/setup-python@v5`](https://github.com/actions/setup-python) with this version. | *(pre-installed Python)* |
| `extras` | pip extras for xonsh, e.g. `full`. Installs as `xonsh[<extras>]`. | *(none)* |
| `xontribs` | **Space-separated** list of extra pip packages (xontribs, etc.) installed after xonsh. E.g. `xontrib-abbrevs xontrib-pipeliner`. | *(none)* |

### Outputs

| Output | Description |
| --- | --- |
| `xonsh-version` | The installed xonsh version (e.g. `0.23.1`). |
| `xonsh-path` | Absolute path to the `xonsh` executable. |

### Cross-platform

Tested on `ubuntu-latest`, `macos-latest`, and `windows-latest`.

### Full example

Uses every input and reads both outputs:

```yaml
jobs:
  my-job:
    runs-on: ubuntu-latest
    defaults:
      run:
        shell: xonsh {0}
    steps:
      - uses: actions/checkout@v4

      - id: xonsh
        uses: xonsh/actions@main
        with:
          python-version: '3.13'         # actions/setup-python before install
          xonsh-version: '0.23.1'        # pinned PyPI version
          extras: 'full'                 # installs xonsh[full]
          xontribs: 'xontrib-abbrevs xontrib-pipeliner'   # space-separated pip packages

      - name: Use xonsh
        run: |
          echo "installed xonsh ${{ steps.xonsh.outputs.xonsh-version }}"
          echo "xonsh binary at ${{ steps.xonsh.outputs.xonsh-path }}"
          print(f"running xonsh {xonsh.__version__} on Python {sys.version.split()[0]}")
          xcontext
          xontrib load pipeliner
          echo hello | pl @!(line.upper())
```

### Sharing state with later steps

`$PATH.append('/mypath')` inside a xonsh step only affects that step's process. To make the change visible in *subsequent* steps, also append to `$GITHUB_PATH`:

```yaml
jobs:
  my-job:
    runs-on: ubuntu-latest
    defaults:
      run:
        shell: xonsh {0}
    steps:
      - uses: actions/checkout@v4
      - uses: xonsh/actions@main

      - run: |
          $PATH.append('/mypath')
          with open($GITHUB_PATH, 'a') as f:
              f.write('/mypath\n')
```

## Reusable workflows

### `test-pip-xontrib.yml`

Real life example from [xontrib-abbrevs/.github/workflows/test.yml](https://github.com/xonsh/xontrib-abbrevs/blob/5fbd8bfaf7cab2ebda05bb8294e6fc14e65f29e5/.github/workflows/test.yml):

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

### `test-poetry-xontrib.yml`

Same matrix, but uses Poetry for dependency management. Pass the `xontrib_name` input so the workflow can `xontrib load` it:

```yaml
jobs:
  testing:
    uses: xonsh/actions/.github/workflows/test-poetry-xontrib.yml@main
    with:
      xontrib_name: my_xontrib
```
