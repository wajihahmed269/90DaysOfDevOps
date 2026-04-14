Day 44 – Secrets, Artifacts & Real Tests
Task 1 & 2: Secrets Workflow
yamlname: Secrets Demo

on: [push]

jobs:
  secrets-demo:
    runs-on: ubuntu-latest
    env:
      DOCKER_USER: ${{ secrets.DOCKER_USERNAME }}
    steps:
      - name: Check secret is set
        run: echo "The secret is set: true"

      - name: Use secret as env var in command
        env:
          MY_SECRET: ${{ secrets.MY_SECRET_MESSAGE }}
        run: |
          echo "Running with secret available in env"
          # Use $MY_SECRET in commands without printing it
When you try to print ${{ secrets.MY_SECRET_MESSAGE }} directly, GitHub replaces the value with *** in the logs — it masks secrets automatically. 
You should never print secrets because CI logs are often accessible to the whole team, get stored, and sometimes leak through third-party integrations.
Task 3 & 4: Artifacts Between Jobs
yamlname: Artifacts Demo

on: [push]

jobs:
  generate:
    runs-on: ubuntu-latest
    steps:
      - name: Generate test report
        run: |
          echo "Test Results" > test-report.txt
          echo "All 42 tests passed" >> test-report.txt
          echo "Coverage: 87%" >> test-report.txt

      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: test-report
          path: test-report.txt

  consume:
    runs-on: ubuntu-latest
    needs: generate
    steps:
      - name: Download artifact
        uses: actions/download-artifact@v4
        with:
          name: test-report

      - name: Read artifact
        run: cat test-report.txt
Real-world artifact uses: passing compiled binaries from build to deploy job, sharing test reports across jobs, storing logs for debugging failed runs, keeping Docker build cache layers.
Task 5 & 6: Real Tests + Caching
yamlname: Real Tests with Cache

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Cache pip dependencies
        uses: actions/cache@v4
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}
          restore-keys: |
            ${{ runner.os }}-pip-

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run tests
        run: python -m pytest tests/ -v
Cache stores the pip download cache directory. First run: no cache hit, downloads everything (~30s). Second run: cache hit, restores instantly (~3s). The cache key is based on requirements.txt hash — if dependencies change, cache is invalidated automatically
