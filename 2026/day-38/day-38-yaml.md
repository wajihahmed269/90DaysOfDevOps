Day 38 – YAML BasicsTask 1 & 2: person.yamlyamlname: Wajih Ahmed
role: DevOps Engineer (Learning)
experience_years: 1
learning: true

tools:
  - Docker
  - Linux
  - Git
  - GitHub Actions
  - AWS

hobbies: [gaming, reading, building projects]Task 3 & 4: server.yamlyamlserver:
  name: prod-server-01
  ip: 192.168.1.10
  port: 8080

database:
  host: db.internal
  name: appdb
  credentials:
    user: admin
    password: supersecret

startup_script: |
  #!/bin/bash
  echo "Starting services..."
  systemctl start nginx
  systemctl start mysql

folded_script: >
  This is a long description
  that will be folded into
  a single line by YAML.Task 5: Validation Notes
yamllint person.yaml — passes clean
When you add a tab instead of spaces, yamllint throws: wrong indentation: expected X but found Y
Fix: replace all tabs with 2 spaces
Task 6: What's Wrong with Block 2yaml# Block 2 - broken
name: devops
tools:
- docker
  - kubernetesProblem: - docker is at the root level (no indentation under tools), then - kubernetes is indented under docker as if it's a child — that's invalid. Both list items must be at the same indentation level under tools.Correct version:
yamltools:
  - docker
  - kubernetesKey Takeaways
YAML uses spaces only — a single tab breaks everything, no exceptions
Two ways to write a list: block style (- item each on new line) and inline style ([item1, item2])
Use | when you need to preserve line breaks (scripts, multiline commands). Use > when whitespace doesn't matter and you want a clean single line output
Strings need quotes only when they contain special characters like : or #
