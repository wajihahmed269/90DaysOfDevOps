Day 46 – Reusable Workflows & Composite Actions
Task 1: Understanding workflow_call

A reusable workflow is a workflow file that doesn't trigger on its own — it exposes a workflow_call trigger so other workflows can call it like a function
The workflow_call trigger means: "this workflow can only be started by another workflow calling it, not by git events"
Calling a reusable workflow (in a jobs: block with uses:) runs an entire separate workflow with its own jobs. Using a regular action (uses: in a step) runs a single step-level action
A reusable workflow must live in .github/workflows/ of the repo it's defined in

Task 2: reusable-build.yml
yamlname: Reusable Build

on:
  workflow_call:
    inputs:
      app_name:
        type: string
        required: true
      environment:
        type: string
        required: true
        default: staging
    secrets:
      docker_token:
        required: true
    outputs:
      build_version:
        description: "Generated build version"
        value: ${{ jobs.build.outputs.build_version }}

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      build_version: ${{ steps.version.outputs.version }}
    steps:
      - uses: actions/checkout@v4

      - name: Print build info
        run: echo "Building ${{ inputs.app_name }} for ${{ inputs.environment }}"

      - name: Confirm secret is set
        run: echo "Docker token is set: true"

      - name: Generate build version
        id: version
        run: |
          SHORT_SHA=$(echo ${{ github.sha }} | cut -c1-7)
          echo "version=v1.0-${SHORT_SHA}" >> $GITHUB_OUTPUT
Task 3 & 4: call-build.yml
yamlname: Call Build Workflow

on:
  push:
    branches: [main]

jobs:
  build:
    uses: ./.github/workflows/reusable-build.yml
    with:
      app_name: "my-web-app"
      environment: "production"
    secrets:
      docker_token: ${{ secrets.DOCKER_TOKEN }}

  print-version:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Print build version
        run: echo "Build version is ${{ needs.build.outputs.build_version }}"
Task 5: Composite Action
.github/actions/setup-and-greet/action.yml:
yamlname: Setup and Greet
description: Greets the user and prints system info

inputs:
  name:
    description: Name to greet
    required: true
  language:
    description: Language for greeting
    required: false
    default: en

outputs:
  greeted:
    description: Whether greeting was done
    value: ${{ steps.greet.outputs.greeted }}

runs:
  using: composite
  steps:
    - name: Print greeting
      id: greet
      shell: bash
      run: |
        if [ "${{ inputs.language }}" = "ur" ]; then
          echo "Salam, ${{ inputs.name }}!"
        else
          echo "Hello, ${{ inputs.name }}!"
        fi
        echo "greeted=true" >> $GITHUB_OUTPUT

    - name: Print system info
      shell: bash
      run: |
        echo "Date: $(date)"
        echo "Runner OS: ${{ runner.os }}"
Workflow using the composite action:
yamlname: Use Composite Action

on: [push]

jobs:
  greet:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run setup and greet
        id: greeter
        uses: ./.github/actions/setup-and-greet
        with:
          name: "Wajih"
          language: "en"

      - name: Check output
        run: echo "Greeted output: ${{ steps.greeter.outputs.greeted }}"
Task 6: Comparison Table
Reusable WorkflowComposite ActionTriggered byworkflow_calluses: in a stepCan contain jobs?YesNo — steps onlyCan contain multiple steps?Yes (inside jobs)YesLives where?.github/workflows/.github/actions/<name>/action.ymlCan accept secrets directly?Yes, 
via secrets: blockNo — secrets must be passed as inputsBest forEntire reusable pipelines (build, test, deploy flows)Reusable step logic within a single job
