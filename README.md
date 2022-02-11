# Server Setup Guide

This is a small guide on setting up my personal home server. The site was built using [mkdocs](https://www.mkdocs.org/) and [squidfunk's mkdocs-material theme](https://github.com/squidfunk/mkdocs-material).

## Development

To develop for this repo you must first create a Python virtual environment:

```text
python3 -m venv ./venv
```

Then activate it, on `unix`:

```text
source ./venv/bin/activate
```

And finally install the dependencies:

```text
pip install -r requirements.txt
```

You can then start a development server with:

```text
mkdocs serve -a 0.0.0.0:8000
```

## Author

This guide repo has been made by [moonstar-x](https://github.com/moonstar-x).
