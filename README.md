# xl2times

`xl2times` is an open source tool to convert TIMES models specified in Excel to a format ready for processing by [GAMS](https://www.gams.com/).
Development of the tool originally started in a Microsoft [repository](https://github.com/microsoft/times-excel-reader) with an intention to make it easier for anyone to reproduce research results on TIMES models.

[TIMES](https://github.com/etsap-TIMES/TIMES_model) is an open source energy systems model generator developed by the [Energy Technology Systems Analysis Program](https://iea-etsap.org/) (ETSAP) of the [International Energy Agency](https://www.iea.org/) (IEA) that is used around the world to inform energy policy.
It is fully explained in the [TIMES Model Documentation](https://iea-etsap.org/index.php/documentation).

Multiple approaches to using spreadsheets for specifying TIMES models have been developed, e.g. [ANSWER-TIMES](https://iea-etsap.org/index.php/etsap-tools/data-handling-shells/answer) and [VEDA-TIMES](https://iea-etsap.org/index.php/etsap-tools/data-handling-shells/veda).
At present, `xl2times` implements partial support of the Veda approach described in the [TIMES Model Documentation PART IV](https://iea-etsap.org/docs/Documentation_for_the_TIMES_Model-Part-IV.pdf) and [Veda Documentation](https://veda-documentation.readthedocs.io/en/latest/pages/VedaTags.html).
Support of other approaches may be added over time.

## Installation and Basic Usage

You can install the latest published version of the tool from PyPI using pip (preferably in a virtual environment):
```bash
pip install xl2times
```

You can also install the latest development version by cloning this repository and running the following command in the root directory:
```bash
pip install .
```

After installation, run the following command to see the basic usage and available options:
```bash
xl2times --help
```

## Documentation

The tool's documentation is at http://xl2times.readthedocs.io/ and the source is in the [`docs/`](https://github.com/etsap-TIMES/xl2times/blob/main/docs) directory.

The documentation is generated by Sphinx and hosted on ReadTheDocs. We use the following extensions:
- `myst-parser`: to be able to write documentation in markdown
- `sphinx-book-theme`: the theme
- `sphinx-copybutton`: to add copy buttons to code blocks
- `sphinxcontrib-apidoc`: to automatically generate API documentation from the Python package

Documentation can be generated locally (after setting up your development environment as described below) by:
```bash
cd docs
make html
```

## Development

### Setup

We recommend installing the tool in editable mode (`-e`) in a Python virtual environment:
```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -U pip
pip install -r requirements.txt
pip install -e .[dev]
```

We use the [black](https://pypi.org/project/black/) code formatter. The `pip` command above will install it along with other requirements.

We also use the [pyright](https://github.com/microsoft/pyright/) type checker -- our GitHub Actions check will fail if pyright detects any type errors in your code. You can install pyright in your virtual environment and check your code by running these commands in the root of the repository:
```bash
pip install pyright==1.1.304
pyright
```
Additionally, you can install a git pre-commit that will ensure that your changes are formatted and pyright detects no issues before creating new commits:
```bash
pre-commit install
```
If you want to skip these pre-commit steps for a particular commit, if for instance pyright has issues but you still want to commit your changes to your branch, you can run:
```bash
git commit --no-verify
```

### Running Benchmarks

See our GitHub Actions CI `.github/workflows/ci.yml` and the utility script `utils/run_benchmarks.py` to see how to run the tool on the DemoS models.

In short, use the commands below to clone the benchmarks data into your local `benchmarks` dir.
Note that this assumes you have access to all these repositories (some are private and
you'll have to request access) - if not, comment out the inaccessible benchmarks from `benchmakrs.yml` before running.

```bash
mkdir benchmarks
# Get TIMES DemoS example models and reference DD files
# XLSX files are in private repo for licensing reasons, please request access or replace with your own files distributed with Veda.
git clone git@github.com:olejandro/demos-xlsx.git benchmarks/xlsx/
git clone git@github.com:olejandro/demos-dd.git benchmarks/dd/

# Get Ireland model and reference DD files
git clone git@github.com:esma-cgep/tim.git benchmarks/xlsx/Ireland
git clone git@github.com:esma-cgep/tim-gams.git benchmarks/dd/Ireland
```
Then to run the benchmarks:
```bash
# Run a only a single benchmark by name (see benchmarks.yml for name list)
python utils/run_benchmarks.py benchmarks.yml --verbose --run DemoS_001-all | tee out.txt

# Run all benchmarks (without GAMS run, just comparing CSV data for regressions)
# Note: if you have multiple remotes, set etsap-TIMES/xl2times as the first one, as it is used for speed/correctness comparisons.
python utils/run_benchmarks.py benchmarks.yml --verbose | tee out.txt


# Run benchmarks with regression tests vs main branch
git branch feature/your_new_changes --checkout
# ... make your code changes here ...
git commit -a -m "your commit message" # code must be committed for comparison to `main` branch to run.
python utils/run_benchmarks.py benchmarks.yml --verbose | tee out.txt
```
At this point, if you haven't broken anything you should see something like:
```
Change in runtime: +2.97s
Change in correct rows: +0
Change in additional rows: +0
No regressions. You're awesome!
```
If you have a large increase in runtime, a decrease in correct rows or fewer rows being produced, then you've broken something and will need to figure out how to fix it.

### Debugging Regressions

If your change is causing regressions on one of the benchmarks, a useful way to debug and find the difference is to run the tool in verbose mode and compare the intermediate tables. For example, if your branch has regressions on Demo 1:
```bash
# First, on the `main` branch:
xl2times benchmarks/xlsx/DemoS_001 --output_dir benchmarks/out/DemoS_001-all --ground_truth_dir benchmarks/csv/DemoS_001-all --verbose > before 2>&1
# Then, on your branch:
git checkout my-branch-name
xl2times benchmarks/xlsx/DemoS_001 --output_dir benchmarks/out/DemoS_001-all --ground_truth_dir benchmarks/csv/DemoS_001-all --verbose > after 2>&1
# And then compare the files `before` and `after`
code -d before after
```
VS Code will highlight the changes in the two files, which should correspond to any differences in the intermediate tables.

### Publishing the Tool

To publish a new version of the tool on PyPI, update the version number in `pyproject.toml`, and then run:
```bash
python -m pip install --upgrade build
python -m pip install --upgrade twine
rm -rf dist
python -m build
python -m twine upload dist/*
```

## Contributing

This project welcomes contributions and suggestions. See [CODE_OF_CONDUCT.md](https://github.com/etsap-TIMES/xl2times/blob/main/CODE_OF_CONDUCT.md) and [CONTRIBUTING.md](https://github.com/etsap-TIMES/xl2times/blob/main/CONTRIBUTING.md) for more details.
