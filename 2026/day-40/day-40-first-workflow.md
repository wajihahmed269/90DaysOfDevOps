Day 40 – First GitHub Actions WorkflowTask 2: hello.ymlyamlname: Hello Workflow

on: [push]

jobs:
  greet:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Say Hello
        run: echo "Hello from GitHub Actions!"Task 4: Updated hello.ymlyamlname: Hello Workflow

on: [push]

jobs:
  greet:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Say Hello
        run: echo "Hello from GitHub Actions!"

      - name: Print current date and time
        run: date

      - name: Print branch name
        run: echo "Branch is ${{ github.ref_name }}"

      - name: List repo files
        run: ls -la

      - name: Print runner OS
        run: echo "Runner OS is ${{ runner.os }}"Task 5: Breaking It on PurposeAdded step:
yaml      - name: Intentional failure
        run: exit 1What happens: the step goes red, all subsequent steps are skipped, the job is marked failed, GitHub sends a notification email, the Actions tab shows a red X. The error in logs shows Process completed with exit code 1. Fixed by removing the step and pushing again — pipeline goes green.Anatomy Notes
on: — defines what event(s) trigger the workflow (push, pull_request, schedule, etc.)
jobs: — groups all the jobs in the workflow; each job runs independently on its own runner
runs-on: — specifies the OS/machine for that job (ubuntu-latest, windows-latest, etc.)
steps: — ordered list of tasks inside a job; runs sequentially top to bottom
uses: — calls a pre-built action from GitHub Marketplace or another repo
run: — executes a raw shell command directly on the runner
name: on a step — just a label for readability in the Actions UI; doesn't affect execution
