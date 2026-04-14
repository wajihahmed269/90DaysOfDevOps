Day 42 – Runners
Task 1: multi-os.yml
yamlname: Multi-OS Runners

on: [push]

jobs:
  ubuntu:
    runs-on: ubuntu-latest
    steps:
      - name: System info
        run: |
          echo "OS: ${{ runner.os }}"
          hostname
          whoami

  windows:
    runs-on: windows-latest
    steps:
      - name: System info
        run: |
          echo "OS: ${{ runner.os }}"
          hostname
          whoami

  macos:
    runs-on: macos-latest
    steps:
      - name: System info
        run: |
          echo "OS: ${{ runner.os }}"
          hostname
          whoami
Task 2: Pre-installed Tools Check
yaml      - name: Check pre-installed tools
        run: |
          docker --version
          python3 --version
          node --version
          git --version
Why it matters: you don't need to install common tools in every job — they're already there, saving pipeline time and setup complexity.
Task 3: Self-Hosted Runner Setup
Steps followed:

Repo → Settings → Actions → Runners → New self-hosted runner
Selected: Linux, x64
Ran the provided script to download, configure, and start the runner
Runner shows as Idle with green dot in GitHub

Task 4: self-hosted.yml
yamlname: Self-Hosted Runner Test

on: [push]

jobs:
  run-locally:
    runs-on: self-hosted
    steps:
      - name: Print hostname
        run: hostname

      - name: Print working directory
        run: pwd

      - name: Create a test file
        run: echo "Runner was here at $(date)" > /tmp/runner-test.txt

      - name: Verify file exists
        run: cat /tmp/runner-test.txt
Task 5: Labels
yaml    runs-on: [self-hosted, my-linux-runner]
Labels are useful when you have multiple self-hosted runners with different specs or purposes — e.g., one runner with GPU access for ML jobs, another lightweight one for basic tests. Labels let workflows target the right machine precisely.
Comparison Table
GitHub-HostedSelf-HostedWho manages it?GitHubYouCostFree up to limits, then paidYour infra costPre-installed toolsMany (Docker, Python, Node, etc.)Only what you installGood forMost standard pipelinesCustom hardware, private networks, cost at scaleSecurity concernShared infrastructure, ephemeralYour machine is persistent — secrets/files can linger between runs

