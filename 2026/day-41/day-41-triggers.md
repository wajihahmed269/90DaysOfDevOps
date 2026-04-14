Day 41 – Triggers & Matrix Builds
Day 41 – Triggers & Matrix Builds
Task 1: pr-check.yml
yamlname: PR Check

on:
  pull_request:
    branches: [main]
    types: [opened, synchronize]

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - name: Print branch name
        run: echo "PR check running for branch: ${{ github.head_ref }}"
Task 2: Scheduled Trigger
Added to any existing workflow:
yamlon:
  schedule:
    - cron: '0 0 * * *'
Cron for every Monday at 9 AM UTC: 0 9 * * 1
Task 3: manual.yml
yamlname: Manual Trigger

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Print environment input
        run: echo "Deploying to ${{ inputs.environment }}"
Task 4 & 5: matrix.yml
yamlname: Matrix Build

on: [push]

jobs:
  build:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        python-version: ["3.10", "3.11", "3.12"]
        os: [ubuntu-latest, windows-latest]
        exclude:
          - os: windows-latest
            python-version: "3.10"

    steps:
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}

      - name: Print Python version
        run: python --version
With 3 Python versions × 2 OS = 6 total jobs, minus 1 excluded = 5 jobs run in parallel.
