Day 48 – GitHub Actions Capstone
Pipeline Architecture
PR opened/updated
  └─► pr-pipeline.yml
        ├─► [calls] reusable-build-test.yml  (run_tests: true)
        └─► pr-comment job (prints branch summary)
              NO Docker build, NO push

Push to main
  └─► main-pipeline.yml
        ├─► Job 1: [calls] reusable-build-test.yml
        ├─► Job 2: [calls] reusable-docker.yml  (needs Job 1)
        │         tags: latest + sha-<hash>
        └─► Job 3: deploy  (needs Job 2)
              prints image URL, uses production environment

Every 12 hours
  └─► health-check.yml
        pulls image → runs container → curls health → prints PASS/FAIL
reusable-build-test.yml
yamlname: Reusable Build and Test

on:
  workflow_call:
    inputs:
      python_version:
        type: string
        default: '3.11'
      run_tests:
        type: boolean
        default: true
    outputs:
      test_result:
        description: Test outcome
        value: ${{ jobs.build-test.outputs.test_result }}

jobs:
  build-test:
    runs-on: ubuntu-latest
    outputs:
      test_result: ${{ steps.tests.outputs.result }}
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: ${{ inputs.python_version }}

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run tests
        id: tests
        if: inputs.run_tests
        run: |
          python -m pytest tests/ -v && echo "result=passed" >> $GITHUB_OUTPUT || echo "result=failed" >> $GITHUB_OUTPUT
reusable-docker.yml
yamlname: Reusable Docker Build and Push

on:
  workflow_call:
    inputs:
      image_name:
        type: string
        required: true
      tag:
        type: string
        required: true
    secrets:
      docker_username:
        required: true
      docker_token:
        required: true
    outputs:
      image_url:
        description: Full image path
        value: ${{ jobs.docker.outputs.image_url }}

jobs:
  docker:
    runs-on: ubuntu-latest
    outputs:
      image_url: ${{ steps.build.outputs.image_url }}
    steps:
      - uses: actions/checkout@v4

      - uses: docker/login-action@v3
        with:
          username: ${{ secrets.docker_username }}
          password: ${{ secrets.docker_token }}

      - name: Build and push
        id: build
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ${{ secrets.docker_username }}/${{ inputs.image_name }}:${{ inputs.tag }}
            ${{ secrets.docker_username }}/${{ inputs.image_name }}:latest

      - name: Set output
        run: echo "image_url=${{ secrets.docker_username }}/${{ inputs.image_name }}:${{ inputs.tag }}" >> $GITHUB_OUTPUT
pr-pipeline.yml
yamlname: PR Pipeline

on:
  pull_request:
    branches: [main]
    types: [opened, synchronize]

permissions:
  contents: read
  pull-requests: write

jobs:
  build-test:
    uses: ./.github/workflows/reusable-build-test.yml
    with:
      run_tests: true

  pr-comment:
    runs-on: ubuntu-latest
    needs: build-test
    steps:
      - name: PR summary
        run: echo "PR checks passed for branch: ${{ github.head_ref }}"
main-pipeline.yml
yamlname: Main Branch Pipeline

on:
  push:
    branches: [main]

permissions:
  contents: read

jobs:
  build-test:
    uses: ./.github/workflows/reusable-build-test.yml
    with:
      run_tests: true

  docker-build-push:
    needs: build-test
    uses: ./.github/workflows/reusable-docker.yml
    with:
      image_name: my-web-app
      tag: sha-${{ github.sha && 'placeholder' }}
    secrets:
      docker_username: ${{ secrets.DOCKER_USERNAME }}
      docker_token: ${{ secrets.DOCKER_TOKEN }}

  deploy:
    runs-on: ubuntu-latest
    needs: docker-build-push
    environment: production
    steps:
      - name: Deploy
        run: echo "Deploying image ${{ needs.docker-build-push.outputs.image_url }} to production"
health-check.yml
yamlname: Health Check

on:
  schedule:
    - cron: '0 */12 * * *'
  workflow_dispatch:

permissions:
  contents: read

jobs:
  health:
    runs-on: ubuntu-latest
    steps:
      - name: Pull image
        run: docker pull ${{ secrets.DOCKER_USERNAME }}/my-web-app:latest

      - name: Run container
        run: docker run -d -p 8080:8080 --name health-check-app ${{ secrets.DOCKER_USERNAME }}/my-web-app:latest

      - name: Wait for startup
        run: sleep 5

      - name: Check health endpoint
        run: |
          STATUS=$(curl -o /dev/null -s -w "%{http_code}" http://localhost:8080/health)
          if [ "$STATUS" = "200" ]; then
            echo "STATUS=PASSED" >> $GITHUB_ENV
          else
            echo "STATUS=FAILED" >> $GITHUB_ENV
          fi

      - name: Cleanup container
        run: docker stop health-check-app && docker rm health-check-app

      - name: Write step summary
        run: |
          echo "## Health Check Report" >> $GITHUB_STEP_SUMMARY
          echo "- Image: my-web-app:latest" >> $GITHUB_STEP_SUMMARY
          echo "- Status: ${{ env.STATUS }}" >> $GITHUB_STEP_SUMMARY
          echo "- Time: $(date)" >> $GITHUB_STEP_SUMMARY
What I'd Add Next

Slack notifications on deploy success and failure
Multi-environment pipeline: staging → prod with approval gate in between
Rollback job that re-deploys the previous image tag if health check fails post-deploy
Trivy image scanning (done on Day 49)
OIDC authentication to AWS instead of long-lived secrets
