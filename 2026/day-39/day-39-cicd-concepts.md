Day 39 – What is CI/CD?Day 39 – CI/CD ConceptsTask 1: The Problem with Manual Deployments
What can go wrong: merge conflicts explode at the last minute, one dev's change breaks another's, someone deploys untested code, rollback is manual and risky, no audit trail of what went where
"Works on my machine" means the app runs fine on the developer's local setup but breaks in production because of different OS versions, missing env variables, different package versions, or different configs — the environments are not identical
Realistically, a team can safely deploy manually maybe 1–2 times a day at best, and only if they're disciplined about it — most teams slip up
Task 2: CI vs CDContinuous Integration (CI)
Every developer merges code to the shared repo frequently — multiple times a day. Each merge automatically triggers a build and test run. It catches integration bugs, broken tests, and conflicts early before they pile up.
Real-world example: A dev pushes a Python change to GitHub, the pipeline runs pytest automatically, and the PR is blocked from merging if tests fail.Continuous Delivery (CD - Delivery)
After CI passes, the code is automatically packaged and made ready to deploy to production — but a human still clicks the final deploy button. "Delivery" means the artifact is always in a deployable state.
Real-world example: After tests pass, a Docker image is built and pushed to a staging environment. The team reviews it, then manually approves the prod deploy.Continuous Deployment (CD - Deployment)
Same as Delivery, but no human approval — every passing build automatically goes to production. Used by teams with high test confidence and mature pipelines.
Real-world example: Netflix or Spotify pushing dozens of small changes to production daily with zero manual steps.Task 3: Pipeline Anatomy
Trigger — the event that starts the pipeline (a git push, a PR, a schedule, a manual button)
Stage — a logical phase grouping related work: build stage, test stage, deploy stage
Job — a unit of work inside a stage that runs on a specific machine, e.g., "run unit tests"
Step — a single command or action inside a job, e.g., pip install -r requirements.txt
Runner — the machine (virtual or physical) that actually executes the job
Artifact — the output produced by a job that can be passed to the next stage, e.g., a compiled binary or Docker image
Task 4: Pipeline Diagram[Developer pushes code to GitHub]
          |
          v
    +------------+
    |   TRIGGER  |  push to main branch
    +------------+
          |
          v
    +------------+
    |    BUILD   |  Install deps, compile, lint
    +------------+
          |
       pass/fail
          |
          v
    +------------+
    |    TEST    |  Unit tests, integration tests
    +------------+
          |
       pass/fail
          |
          v
    +------------+
    |   DOCKER   |  Build image, tag, push to registry
    +------------+
          |
          v
    +------------+
    |   DEPLOY   |  Pull image on staging server, run container
    +------------+
          |
          v
    [App live on staging]Task 5: Open-Source Repo ExplorationExplored: FastAPI (tiangolo/fastapi)
Trigger: push to master, pull_request to master
Jobs: about 5 jobs — lint, test across multiple Python versions (3.8 to 3.12), coverage
What it does: runs mypy for type checking, ruff for linting, pytest for tests, and uploads coverage to Codecov — a solid real-world example of matrix testing across Python versions
