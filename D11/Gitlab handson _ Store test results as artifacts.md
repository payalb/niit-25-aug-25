<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# Gitlab handson : Store test results as artifacts

Provide detailed step by step instruction

Here are **detailed, step-by-step instructions to store test results as artifacts in GitLab CI/CD**. This guide uses a sample Python project with PyTest, but the steps are almost identical for other languages and test tools.

## Step 1: Ensure Your Test Command Generates a Report

- Most test frameworks can generate files (e.g., JUnit XML, HTML, etc.) summarizing test results.
- For PyTest, use:

```sh
pytest --junitxml=report.xml
```

- For Java/JUnit, typically:

```sh
mvn test
# JUnit creates XML files under target/surefire-reports/
```


## Step 2: Create or Edit `.gitlab-ci.yml`

- In the root of your project, there should be a `.gitlab-ci.yml` file.
- If it doesn’t exist, create it.
- Add (or update) a `job` that runs your tests and saves the test result file.


### Example for a Python Project

```yaml
stages:
  - test

pytest:
  stage: test
  image: python:3.11
  script:
    - pip install -r requirements.txt
    - pytest --junitxml=report.xml
  artifacts:
    paths:
      - report.xml
    expire_in: 1 week  # Optional: retains artifact for 1 week
```


### Example for a NodeJS Project

```yaml
stages:
  - test

npm-test:
  stage: test
  image: node:20
  script:
    - npm install
    - npm test -- --reporters=junit --reporter-options output=report.xml
  artifacts:
    paths:
      - report.xml
```


## Step 3: Push Your Changes to GitLab

1. Commit your `.gitlab-ci.yml` changes:

```sh
git add .gitlab-ci.yml
git commit -m "Add test artifacts job"
git push
```

2. This triggers a pipeline run on GitLab.

## Step 4: Monitor the Pipeline

- Go to your repository on GitLab.
- Click on **CI/CD > Pipelines**.
- Click the current pipeline to follow job progress.


## Step 5: Download the Test Report as Artifact

- Once the job completes, click on the test job in the GitLab UI (e.g., `pytest`, `npm-test`).
- On the right, there will be an **Artifacts** section.
- Click `report.xml` to **download** the test report.
- Artifacts can also be referenced by later jobs in the same pipeline, if you need to use them for further processing (e.g., test summary, deployment).


## Tips

- You can specify multiple files or folders in `artifacts.paths`.
- Use the `expire_in:` option to control how long artifacts are retained.
- For multiple test files, point to a directory, e.g.,

```yaml
artifacts:
  paths:
    - test-results/
```

- Job artifacts are automatically deleted after the expiration time, helping manage storage.

By following these steps, you'll reliably store and access your test results as artifacts in GitLab CI/CD, enabling better debugging, auditing, and reporting across your team.

