Day 49 – DevSecOps
What DevSecOps Means
DevSecOps means security is no longer a final checkpoint before release — it's automated and runs on every PR and every push, the same way tests do. Instead of a security team reviewing code weeks after it's written, the pipeline catches vulnerabilities, exposed secrets, and risky dependencies the moment code is pushed, when fixing them costs minutes instead of days.
Task 1: Trivy Image Scan (added to main-pipeline.yml after Docker build)
yaml      - name: Scan Docker Image for Vulnerabilities
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: '${{ secrets.DOCKER_USERNAME }}/my-web-app:latest'
          format: 'table'
          exit-code: '1'
          severity: 'CRITICAL,HIGH'
Notes: Base image matters enormously here. python:3.11 has significantly fewer CVEs than python:3.11-slim which has fewer than ubuntu:20.04. If Trivy fails your pipeline, the immediate fix is to switch to a minimal base image like python:3.11-alpine or a distroless image. Common CVEs found are in system packages like libssl, libc, or zlib bundled into the base OS layer — these are not your code's vulnerabilities, they're the base image's baggage.
Task 2: GitHub Secret Scanning Notes
Secret scanning detects secrets already committed to the repo — GitHub scans your history and alerts you if it finds API keys, tokens, or passwords.
Push protection is proactive — it blocks the push entirely before the secret ever lands in the repo.
If GitHub detects a leaked AWS key: it notifies the repo owner via email, notifies AWS directly (AWS revokes the key automatically in many cases), and flags the commit in the Security tab. The key is compromised from the moment it was pushed even if you delete the commit — git history is not private and the key must be rotated immediately.
Task 3: Dependency Review (added to pr-pipeline.yml)
yaml  dependency-review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4
      - uses: actions/dependency-review-action@v4
        with:
          fail-on-severity: critical
Task 4: Permissions Blocks
Added to pr-pipeline.yml and main-pipeline.yml:
yamlpermissions:
  contents: read
For workflows that comment on PRs:
yamlpermissions:
  contents: read
  pull-requests: write
Why limit permissions: workflows run arbitrary code from the internet (third-party actions). If a third-party action is compromised (supply chain attack), it runs inside your workflow with whatever permissions you gave it. If you gave it contents: write, it can push malicious commits to your repo. If you limit it to contents: read, the blast radius is contained. Principle of least privilege — give only what's needed.
Final Secure Pipeline Diagram
PR opened
  ├─► build & test
  ├─► dependency vulnerability check   (fails on CRITICAL CVE in new deps)
  └─► PR checks pass or fail

Push to main
  ├─► build & test
  ├─► Docker build
  ├─► Trivy image scan                 (fails on CRITICAL/HIGH CVE)
  ├─► Docker push                      (only if scan passes)
  └─► deploy to production

Always active (no workflow needed)
  ├─► GitHub secret scanning           (alerts on leaked secrets in history)
  └─► Push protection                  (blocks push if secret detected)

Every 12 hours
  └─► health check   
