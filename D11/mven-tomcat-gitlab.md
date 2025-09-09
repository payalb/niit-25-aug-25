<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# image: maven:3.8.6-openjdk-17

services:

- docker:dind

variables:
DOCKER_DRIVER: overlay2

before_script:

- apt-get update \&\& apt-get install -y docker.io

stages:

- build
- deploy

build:
stage: build
script:
- git clone https://github.com/Vishwanathms/SampleMaven.git
- cd SampleMaven
- mvn clean package
artifacts:
paths:
- SampleMaven/target/*.jar
expire_in: 1 day

deploy:
stage: deploy
script:
- cd SampleMaven
- echo "Creating Dockerfile"
- |
cat <<EOF > Dockerfile
FROM openjdk:17
COPY target/*.jar app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
EOF
- docker build -t sample-app .
- docker run -d -p 8080:8080 --name sample-app sample-app

eplain the code properly

Here’s a detailed, step-by-step explanation of your GitLab CI/CD pipeline YAML:

## 1. Image Selection

```yaml
image: maven:3.8.6-openjdk-17
```

- All pipeline jobs run in a pre-built Docker image containing Maven 3.8.6 and OpenJDK 17.
- Ensures Maven and JDK are always available, delivering a clean, consistent build environment for Java projects.


## 2. Services: Docker-in-Docker (DinD)

```yaml
services:
  - docker:dind
```

- Runs the `docker:dind` (Docker-in-Docker) service alongside your main job container.
- Enables you to run Docker commands (`docker build`, `docker run`) inside a job—crucial for Docker-based build/deploy tasks.


## 3. Variables

```yaml
variables:
  DOCKER_DRIVER: overlay2
```

- Sets the Docker storage driver to `overlay2`, which is the recommended, efficient option for modern Linux-based containers.


## 4. Before Script

```yaml
before_script:
  - apt-get update && apt-get install -y docker.io
```

- Updates the package index and installs Docker CLI inside the Maven image.
- Required because the base Maven image generally **does not** include the Docker CLI tools by default.
- With DinD (docker:dind service), this lets you issue Docker commands from within the job.


## 5. Stages

```yaml
stages:
  - build
  - deploy
```

- Declares the pipeline stages:
    - **build:** Compiles and packages your application.
    - **deploy:** Packages the app into a Docker container and launches it.


## 6. Build Job

```yaml
build:
  stage: build
  script:
    - git clone https://github.com/Vishwanathms/SampleMaven.git
    - cd SampleMaven
    - mvn clean package
  artifacts:
    paths:
      - SampleMaven/target/*.jar
    expire_in: 1 day
```

**What happens:**

- Clones the SampleMaven Java project from GitHub.
- Changes directory into the project folder.
- Runs `mvn clean package` to produce the application JAR (outputs to `target/*.jar`).
- **Artifacts:** Stores the built JAR as a pipeline artifact, making it available to the next job (and downloadable from the UI) for 1 day.


## 7. Deploy Job

```yaml
deploy:
  stage: deploy
  script:
    - cd SampleMaven
    - echo "Creating Dockerfile"
    - |
      cat <<EOF > Dockerfile
      FROM openjdk:17
      COPY target/*.jar app.jar
      ENTRYPOINT ["java", "-jar", "/app.jar"]
      EOF
    - docker build -t sample-app .
    - docker run -d -p 8080:8080 --name sample-app sample-app
```

**What happens step-by-step:**

1. Enters the project directory containing the already-built JAR.
2. Generates a Dockerfile on the fly with these commands:
    - **FROM openjdk:17:** Uses the OpenJDK 17 image as a base.
    - **COPY target/*.jar app.jar:** Copies the packaged JAR into the container as `app.jar`.
    - **ENTRYPOINT:** Runs the JAR using the standard Java command.
3. **docker build -t sample-app .**
    - Builds a Docker image tagged as `sample-app` containing your Java application.
4. **docker run -d -p 8080:8080 --name sample-app sample-app**
    - Runs the container in detached mode, exposing port 8080.
    - Names the running instance `sample-app`.

## Summary Table

| Section | Purpose |
| :-- | :-- |
| image | Set Maven+JDK environment for all jobs |
| services | Enables Docker commands via Docker-in-Docker |
| variables | Sets storage driver for Docker |
| before_script | Installs Docker CLI |
| build job | Clones code, builds JAR, saves as artifact |
| deploy job | Builds Docker image, runs container with your Java app |

**Key Points:**

- **Both build and deploy are automated:** you compile, package, Dockerize, and run your app via CI.
- **Artifacts bridge build/deploy jobs**, providing a handoff of the JAR file.
- **Docker-in-Docker plus CLI install** allow true container builds within CI.
- **Everything runs “on the fly”:** no manual steps, all configuration lives in the YAML.

Let me know if you need clarification on any phase or want to expand to include push-to-registry or deployment to cloud!

