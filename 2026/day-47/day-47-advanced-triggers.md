Day 47 – Advanced Triggers
Task 1: pr-lifecycle.yml
yamlname: PR Lifecycle

on:
  pull_request:
    types: [opened, synchronize, reopened, closed]

jobs:
  lifecycle:
    runs-on: ubuntu-latest
    steps:
      - name: Print PR event details
        run: |
          echo "Event type: ${{ github.event.action }}"
          echo "PR title: ${{ github.event.pull_request.title }}"
          echo "PR author: ${{ github.event.pull_request.user.login }}"
          echo "Source branch: ${{ github.head_ref }}"
          echo "Target branch: ${{ github.base_ref }}"

      - name: Merged notification
        if: github.event.action == 'closed' && github.event.pull_request.merged == true
        run: echo "PR was merged into ${{ github.base_ref }}"
Task 2: pr-checks.yml
yamlname: PR Validation Checks

on:
  pull_request:
    branches: [main]

jobs:
  file-size-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Check file sizes
        run: |
          find . -type f -size +1M | while read file; do
            echo "FAIL: $file exceeds 1MB"
            exit 1
          done
          echo "All files within size limit"

  branch-name-check:
    runs-on: ubuntu-latest
    steps:
      - name: Validate branch name
        run: |
          BRANCH="${{ github.head_ref }}"
          if echo "$BRANCH" | grep -qE '^(feature|fix|docs)/'; then
            echo "Branch name valid: $BRANCH"
          else
            echo "Branch name '$BRANCH' must start with feature/, fix/, or docs/"
            exit 1
          fi

  pr-body-check:
    runs-on: ubuntu-latest
    steps:
      - name: Check PR description
        run: |
          BODY="${{ github.event.pull_request.body }}"
          if [ -z "$BODY" ]; then
            echo "WARNING: PR description is empty. Please describe your changes."
          else
            echo "PR description present"
          fi
Task 3: scheduled-tasks.yml
yamlname: Scheduled Tasks

on:
  schedule:
    - cron: '30 2 * * 1'
    - cron: '0 */6 * * *'
  workflow_dispatch:

jobs:
  health-check:
    runs-on: ubuntu-latest
    steps:
      - name: Print which schedule triggered
        run: echo "Triggered by schedule: ${{ github.event.schedule }}"

      - name: Health check
        run: |
          STATUS=$(curl -o /dev/null -s -w "%{http_code}" https://example.com)
          echo "Response code: $STATUS"
          if [ "$STATUS" != "200" ]; then
            echo "Health check FAILED"
            exit 1
          fi
          echo "Health check PASSED"
Cron expressions:

Every weekday at 9 AM IST (IST = UTC+5:30, so 9 AM IST = 3:30 AM UTC): 30 3 * * 1-5
First day of every month at midnight UTC: 0 0 1 * *

Why scheduled workflows get delayed: GitHub deprioritizes scheduled workflows on repos with no recent activity to save compute. If nobody has pushed or interacted with the repo in a while, GitHub may skip or delay the schedule entirely. The fix: trigger workflow_dispatch manually to keep it active, or accept the behavior for non-critical schedules.
Task 4: smart-triggers.yml
yamlname: Smart Triggers

on:
  push:
    branches: [main, 'release/*']
    paths:
      - 'src/**'
      - 'app/**'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Source code changed — running build"
yamlname: Skip Docs Changes

on:
  push:
    paths-ignore:
      - '*.md'
      - 'docs/**'

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Non-doc change detected — running tests"
Use paths when you only want to run on specific file changes (positive filter — narrow the trigger). Use paths-ignore when you want to run on everything EXCEPT certain files 
(negative filter — broader trigger with exclusions). paths is better when your repo has distinct modules with separate pipelines. paths-ignore is better when doc or config changes 
genuinely never need a build/test run.
Task 5: workflow_run Chain
tests.yml:
yamlname: Run Tests

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "Running tests..."
      - run: exit 0
deploy-after-tests.yml:
yamlname: Deploy After Tests

on:
  workflow_run:
    workflows: ["Run Tests"]
    types: [completed]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Check test result
        run: |
          if [ "${{ github.event.workflow_run.conclusion }}" != "success" ]; then
            echo "Tests failed — skipping deploy"
            exit 1
          fi
          echo "Tests passed — deploying"
Task 6: repository_dispatch
yamlname: External Trigger

on:
  repository_dispatch:
    types: [deploy-request]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Print payload
        run: echo "Deploying to ${{ github.event.client_payload.environment }}"
Trigger with:
bashgh api repos/<owner>/<repo>/dispatches \
  -f event_type=deploy-request \
  -f client_payload='{"environment":"production"}'
When would an external system trigger a pipeline: a Slack bot with a /deploy production slash command, a monitoring tool that detects an anomaly and triggers a rollback pipeline, a ticketing system that auto-deploys
when a release ticket is approved, or an external webhook from a partner service signaling data is ready to process.
workflow_run vs workflow_call
workflow_call: synchronous, the caller waits — you get a parent-child relationship within the same run, outputs flow back, secrets can be passed. 
Use it when you want modular reusable pipelines that are part of the same logical run.
workflow_run: asynchronous, event-based — Workflow B fires only after Workflow A completes. No shared context, no output passing. Use it when workflows are logically separate (tests are their own thing, deploy is its own thing) and you want loose coupling between them.
