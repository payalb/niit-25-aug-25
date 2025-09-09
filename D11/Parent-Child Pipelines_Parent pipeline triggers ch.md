<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# Parent-Child Pipelines

Parent pipeline triggers child pipeline using trigger: keyword.
Example:
yaml
CopyEdit

# .gitlab-ci.yml (Parent Pipeline)

stages:

- trigger

trigger-child:
stage: trigger
trigger:
include: child-pipeline.yml
strategy: depend

yaml
CopyEdit

# child-pipeline.yml (Child Pipeline)

stages:

- build
- test

child-build:
stage: build
script: echo "Child build"

child-test:
stage: test
script: echo "Child test"

🔹 Behavior:
The parent triggers the child-pipeline.yml file as a separate pipeline.

Here is a structured slide you can use for your PowerPoint presentation explaining **Parent-Child Pipelines** in CI/CD systems, with your provided YAML examples:

### Parent-Child Pipelines

**Concept:**
A *parent pipeline* can trigger an external *child pipeline* using the `trigger:` keyword, allowing for modular, reusable, or split pipeline configurations.

#### Example

**Parent Pipeline (`.gitlab-ci.yml`):**

```yaml
stages:
  - trigger

trigger-child:
  stage: trigger
  trigger:
    include: child-pipeline.yml
    strategy: depend
```

**Child Pipeline (`child-pipeline.yml`):**

```yaml
stages:
  - build
  - test

child-build:
  stage: build
  script: echo "Child build"

child-test:
  stage: test
  script: echo "Child test"
```


#### Behavior

- The parent pipeline uses the `trigger:` keyword to initiate the child pipeline (`child-pipeline.yml`).
- The child pipeline runs its stages (`build`, `test`) independently, as a separate pipeline, but is managed and monitored by the parent.
- The `strategy: depend` ensures the parent waits for the child to complete and reports its result (success/failure) in the parent pipeline status.

You can visualize this as two blocks:

- Parent pipeline (triggers child)
    - ↓
- Child pipeline (runs its own jobs/stages and reports back to parent)

This structure helps with pipeline modularity, division of responsibility, and reusability across projects or components.

