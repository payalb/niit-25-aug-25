<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# gitlab handson: Integrating unit testing tools (e.g., JUnit, PyTest)

Provide detailed step by step instruction

Here are **detailed, step-by-step instructions to integrate unit testing tools (PyTest for Python, JUnit for Java) with GitLab CI/CD**. These steps help you automate testing, store results, and get feedback directly in merge requests or pipelines.

## Step 1: Add/Verify Unit Tests in Your Project

- **Python:** Place your tests (e.g., `test_*.py`) in a `tests/` folder or alongside your code.
- **Java:** Use the standard `src/test/java/` directory for JUnit tests.


## Step 2: Ensure Test Tools are in Your `requirements.txt`/`pom.xml`

- **Python:**
Add to `requirements.txt`:

```
pytest
pytest-cov  # (for coverage reports, optional)
```

- **Java (Maven):**
Ensure `pom.xml` has:

```xml
<dependency>
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <version>4.13.2</version>
  <scope>test</scope>
</dependency>
```


## Step 3: Create/Edit `.gitlab-ci.yml` in Project Root

### For Python (PyTest):

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
    expire_in: 1 week
```

- `pytest --junitxml=report.xml` creates a JUnit-compatible test results file.
- `artifacts:` uploads it as a downloadable artifact in GitLab UI.


### For Java (JUnit+Maven):

```yaml
stages:
  - test

junit-test:
  stage: test
  image: maven:3.9-jdk-17
  script:
    - mvn test
  artifacts:
    paths:
      - target/surefire-reports/
    expire_in: 1 week
```

- Maven's `mvn test` generates XML reports inside `target/surefire-reports/`.


## Step 4: Commit \& Push Your Changes

```sh
git add .gitlab-ci.yml
git commit -m "Setup CI: integrate unit tests"
git push
```

- This triggers a pipeline. Each push will now run your tests automatically.


## Step 5: Review the Pipeline in GitLab

- Navigate to **CI/CD > Pipelines**.
- Click on your pipeline, then the test job.
- If tests pass, the job will be marked **success**; if tests fail, the pipeline/job fails.


## Step 6: Download or View Test Reports

- Inside the job log, look for an **Artifacts** section.
- Download `report.xml` (or JUnit XML for Java) for details on each test run.
- Optionally, add further jobs to process artifacts or notify your team based on results.


### Optional: Advanced Integration

- Add code coverage reports, HTML output, or advanced reporters.
    - **PyTest:** Add `--cov=my_package --cov-report=xml`
    - **Java (JaCoCo):** Add the JaCoCo plugin in Maven for coverage.
- Fail the pipeline if coverage drops below threshold using plugins or extra jobs.
- Use build badges in your README for passing/failing pipelines and coverage.


## Troubleshooting Tips

- Check job logs for errors—missing dependencies, typo in artifact paths, etc.
- Ensure the Docker image used has all tooling (Python/Java, pip/maven).
- If reports aren’t showing, verify the relative path to your results in `artifacts.paths`.

By following these instructions, you'll automate the process of testing your code in GitLab, capturing test results as artifacts for review and team collaboration, and ensure better code quality through continuous integration!

