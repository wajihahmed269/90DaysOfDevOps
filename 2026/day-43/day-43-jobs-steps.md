Day 43 – Jobs, Steps, Env Vars & Conditionals
Task 1: multi-job.yml
yamlname: Multi-Job Pipeline

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Build
        run: echo "Building the app"

  test:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Test
        run: echo "Running tests"

  deploy:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - name: Deploy
        run: echo "Deploying"
Task 2: Environment Variables
yamlname: Env Vars Demo

on: [push]

env:
  APP_NAME: myapp

jobs:
  show-vars:
    runs-on: ubuntu-latest
    env:
      ENVIRONMENT: staging
    steps:
      - name: Print all vars
        env:
          VERSION: 1.0.0
        run: |
          echo "App: $APP_NAME"
          echo "Environment: $ENVIRONMENT"
          echo "Version: $VERSION"
          echo "Commit SHA: ${{ github.sha }}"
          echo "Triggered by: ${{ github.actor }}"
Task 3: Job Outputs
yamlname: Job Outputs

on: [push]

jobs:
  generate:
    runs-on: ubuntu-latest
    outputs:
      today: ${{ steps.set-date.outputs.date }}
    steps:
      - name: Set date output
        id: set-date
        run: echo "date=$(date +'%Y-%m-%d')" >> $GITHUB_OUTPUT

  consume:
    runs-on: ubuntu-latest
    needs: generate
    steps:
      - name: Use date from previous job
        run: echo "Date from Job 1 was ${{ needs.generate.outputs.today }}"
Why pass outputs between jobs: jobs run on separate runners with no shared memory. If Job 1 builds an artifact version number or generates a config, Job 2 needs a formal handoff mechanism to use that data. Outputs are that mechanism.
Task 4 & 5: smart-pipeline.yml
yamlname: Smart Pipeline

on: [push]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - name: Lint
        run: echo "Linting code..."

      - name: Only on main
        if: github.ref == 'refs/heads/main'
        run: echo "This is a main branch push"

      - name: Risky step
        continue-on-error: true
        run: echo "This might fail but pipeline continues"

      - name: Runs only if previous step failed
        if: failure()
        run: echo "Previous step failed — alerting team"

  test:
    runs-on: ubuntu-latest
    if: github.event_name == 'push'
    steps:
      - name: Test
        run: echo "Running tests..."

  summary:
    runs-on: ubuntu-latest
    needs: [lint, test]
    steps:
      - name: Print summary
        run: |
          if [ "${{ github.ref_name }}" = "main" ]; then
            echo "Main branch push — production pipeline"
          else
            echo "Feature branch push: ${{ github.ref_name }}"
          fi
          echo "Commit message: ${{ github.event.commits[0].message }}
