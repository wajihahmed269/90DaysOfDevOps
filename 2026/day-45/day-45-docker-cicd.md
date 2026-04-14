Day 45 – Docker CI/CD
docker-publish.yml
yamlname: Docker Build and Push

on:
  push:
    branches: [main]

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_TOKEN }}

      - name: Get short SHA
        id: sha
        run: echo "short=$(echo ${{ github.sha }} | cut -c1-7)" >> $GITHUB_OUTPUT

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: ${{ github.ref == 'refs/heads/main' }}
          tags: |
            ${{ secrets.DOCKER_USERNAME }}/my-web-app:latest
            ${{ secrets.DOCKER_USERNAME }}/my-web-app:sha-${{ steps.sha.outputs.short }}
Task 4: Only Push on Main
The push: ${{ github.ref == 'refs/heads/main' }} line handles this. On feature branches, the image builds (good for catching Dockerfile errors) but is never pushed to Docker Hub.
Task 5: README Badge
markdown![Docker Build](https://github.com/<your-username>/github-actions-practice/actions/workflows/docker-publish.yml/badge.svg)
Full Journey: git push → Running Container

git push origin main — triggers the workflow via the on: push event
GitHub spins up a fresh ubuntu-latest runner
actions/checkout pulls your code onto the runner
docker/login-action authenticates to Docker Hub using your secrets
docker/build-push-action runs docker build on your Dockerfile
The built image is tagged as latest and sha-<hash> then pushed to Docker Hub
Anyone (or any server) can now run: docker pull yourname/my-web-app:latest && docker run -p 8080:8080 yourname/my-web-app:latest
Container is live
