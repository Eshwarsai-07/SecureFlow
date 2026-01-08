# Chapter 4: Automated CI/CD Pipeline

In the previous chapter, [Frontend Development & Build Tooling](03_frontend_development___build_tooling_.md), we learned how to use tools like Vite, TypeScript, ESLint, and Tailwind CSS to build, check, and style our **SecureFlow** Tic Tac Toe game. We saw how `npm run build` transforms our development code into a ready-to-deploy website.

Now, imagine you're a chef who has perfected a delicious recipe. You've prepared the ingredients, followed the steps, and baked a perfect cake. But what if you have to bake that cake *every single time* someone asks for it, and you also need to make sure it's always perfect, no matter who baked it? That would be a lot of manual work!

This is where an **Automated CI/CD Pipeline** comes in for our code. It's like setting up a fully automated bakery. Once you have the recipe (your code), the bakery (the pipeline) automatically handles all the steps: mixing ingredients, baking, quality control, packaging, and even delivering it to the customer – all consistently, reliably, and without you having to do it manually every time!

### The Automated Factory: What Problem It Solves

In our **SecureFlow** project, we want to make sure that every time a developer makes changes to our Tic Tac Toe game and saves them, the following things happen *automatically*:

1.  **Testing**: Is the game logic still working correctly? (No new bugs?)
2.  **Quality Check**: Does the code follow our team's style guides? (Is it neat?)
3.  **Building**: Can we successfully create the final web application?
4.  **Packaging**: Can we put our application into a portable "box" (a Docker container) so it can run anywhere?
5.  **Security Scan**: Is our packaged application safe from known vulnerabilities?
6.  **Deployment Prep**: Can we automatically tell our servers to use this new, updated, and secure version of the game?

Doing all these steps manually every time would be slow, error-prone, and frustrating. The problem an **Automated CI/CD Pipeline** solves is: **How do we ensure consistent quality, security, and efficient delivery of our game to players, every single time code changes, without manual intervention?**

We solve this using **GitHub Actions**, which acts as our automated factory conveyor belt.

### Key Concepts: Our Automated Factory Stations

Our CI/CD pipeline is built using **GitHub Actions**, which is a powerful tool integrated right into GitHub. Think of it as the control system for our automated factory.

Here are the key concepts and what they do in our factory:

| Concept Name       | Analogy                                   | What it does for SecureFlow                                     |
| :----------------- | :---------------------------------------- | :-------------------------------------------------------------- |
| **CI (Continuous Integration)** | The "Quality Control" section of the factory | Ensures new code integrates smoothly: tests, linting, building. |
| **CD (Continuous Delivery/Deployment)** | The "Packaging & Delivery" section    | Prepares the app for release: Docker packaging, security scan, deployment update. |
| **GitHub Actions** | The automated control system              | The platform that runs our pipeline.                            |
| **`ci-cd.yml`**    | The factory's "instruction manual"        | A YAML file that tells GitHub Actions exactly what steps to run and in what order. |

---

### How Our Automated Factory Works (The Conveyor Belt)

In our `SecureFlow` project, whenever you push new changes to the `main` branch (or create a pull request to merge changes into `main`), our `ci-cd.yml` instruction manual tells GitHub Actions to start the automated factory.

Here's the sequence of operations, like items moving along a conveyor belt:

1.  **Trigger**: Someone (maybe you!) pushes new code to the `main` branch on GitHub.
2.  **Start Workflow**: GitHub sees the push and, thanks to `ci-cd.yml`, it starts a "workflow" (our automated pipeline).
3.  **Unit Testing** (CI Stage 1): The code first goes to the "Testing Station." Here, `Vitest` (from [Game Logic Engine](02_game_logic_engine_.md)) runs all our game's tests to make sure everything still works. If tests fail, the belt stops!
4.  **Static Code Analysis** (CI Stage 2): Next, it moves to the "Code Quality Station." `ESLint` (from [Frontend Development & Build Tooling](03_frontend_development___build_tooling_.md)) checks our code for style and common errors. If the code is messy, the belt might warn us or stop.
5.  **Build** (CI Stage 3): If the tests pass and code quality is good, it goes to the "Assembly Station." `Vite` (from [Frontend Development & Build Tooling](03_frontend_development___build_tooling_.md)) takes our source code and builds the final, optimized web application.
6.  **Docker Build & Scan** (CD Stage 1): The assembled application is now "packaged." It's put into a **Docker container** (like a sealed, portable box for our app, which we'll learn about in [Docker Containerization](05_docker_containerization_.md)). Then, `Trivy` scans this Docker container for any security vulnerabilities. If vulnerabilities are found, the belt stops!
7.  **Docker Push** (CD Stage 2): If the Docker container is safe, it's pushed to a central storage place called a **Container Registry** (like a warehouse for our app boxes).
8.  **Update Kubernetes Deployment** (CD Stage 3): Finally, the last station on the belt "informs the delivery truck." It updates a special file (`kubernetes/deployment.yaml`) to tell our server management system (**Kubernetes**, which we'll cover in [Kubernetes Deployment & Orchestration](06_kubernetes_deployment___orchestration_.md)) that there's a new, updated, and secure version of our game available.

This entire process ensures that every change goes through a rigorous quality and security check, and then gets delivered efficiently.

### Under the Hood: Our `ci-cd.yml` Instruction Manual

All of this automation is defined in a special file: `.github/workflows/ci-cd.yml`. This YAML file is the step-by-step instruction manual for GitHub Actions.

Let's look at simplified parts of this file to understand how it orchestrates our automated factory.

#### 1. Triggering the Workflow

First, we tell GitHub Actions *when* to start the pipeline:

```yaml
# .github/workflows/ci-cd.yml (Simplified)
name: CI/CD Pipeline

on:
  push:
    branches: [ main ] # Run when code is pushed to the 'main' branch
    paths-ignore:
      - 'kubernetes/deployment.yaml' # Don't trigger if ONLY this file changes
  pull_request:
    branches: [ main ] # Run when a pull request targets 'main'
```

**Explanation**: This section says, "Hey GitHub, please run this pipeline whenever someone pushes code to the `main` branch, OR when someone creates a `pull_request` to merge code into the `main` branch." We also add a clever rule `paths-ignore` to prevent the pipeline from running forever if the last step (updating `kubernetes/deployment.yaml`) causes another `push`.

#### 2. The Jobs: Our Factory Stations

Each major step in our factory is called a `job`. Jobs can run in parallel or in sequence (one after another).

##### Job 1: `test` (Unit Testing Station)

```yaml
# .github/workflows/ci-cd.yml (Simplified test job)
jobs:
  test:
    name: Unit Testing
    runs-on: ubuntu-latest # Run this job on a fresh Ubuntu virtual machine
    steps:
      - name: Checkout code
        uses: actions/checkout@v4 # Get our code from the repository
      - name: Setup Node.js
        uses: actions/setup-node@v4 # Install Node.js
        with:
          node-version: '20'
      - name: Install dependencies
        run: npm ci # Install project dependencies (like React, Vitest)
      - name: Run tests
        run: npm test # Execute our game logic tests (from Chapter 2)
```

**Explanation**: This job sets up a clean environment, gets our project code, installs necessary tools (`Node.js`), installs our project's dependencies (`npm ci`), and then runs `npm test`. If any tests fail here, the entire pipeline stops, preventing faulty code from moving forward.

##### Job 2: `lint` (Code Quality Station)

```yaml
# .github/workflows/ci-cd.yml (Simplified lint job)
  lint:
    name: Static Code Analysis
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      - name: Install dependencies
        run: npm ci
      - name: Run ESLint
        run: npm run lint # Execute our code quality checks (from Chapter 3)
```

**Explanation**: Similar to `test`, this job sets up the environment and then runs `npm run lint`. This checks our code against the `ESLint` rules we configured in [Frontend Development & Build Tooling](03_frontend_development___build_tooling_.md), ensuring code consistency and catching potential errors.

##### Job 3: `build` (Assembly Station)

```yaml
# .github/workflows/ci-cd.yml (Simplified build job)
  build:
    name: Build
    runs-on: ubuntu-latest
    needs: [test, lint] # ONLY run this job if 'test' AND 'lint' jobs passed
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      - name: Install dependencies
        run: npm ci
      - name: Build project
        run: npm run build # Create the final web application (from Chapter 3)
      - name: Upload build artifacts
        uses: actions/upload-artifact@v4 # Save the built app for later jobs
        with:
          name: build-artifacts
          path: dist/
```

**Explanation**: The `needs: [test, lint]` line is very important! It tells GitHub Actions: "Don't even start building until *both* the `test` and `lint` jobs have successfully completed." This ensures we only build good quality code. After building with `npm run build`, we use `actions/upload-artifact` to save the built game files (`dist/` folder) so other jobs (like the Docker job) can use them.

##### Job 4: `docker` (Packaging & Security Scan Station)

```yaml
# .github/workflows/ci-cd.yml (Simplified docker job)
  docker:
    name: Docker Build and Push
    runs-on: ubuntu-latest
    needs: [build] # ONLY run if the 'build' job passed
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      - name: Download build artifacts
        uses: actions/download-artifact@v4 # Get the built app
        with:
          name: build-artifacts
          path: dist/
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      - name: Login to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          username: ${{ github.actor }}
          password: ${{ secrets.TOKEN }} # Use GitHub's secret key
      - name: Build Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: false # Build it locally first
          # ... simplified tags and labels ...
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master # Scan the Docker image for security issues
        with:
          # ... image reference and severity settings ...
          exit-code: '1' # Fail pipeline if CRITICAL/HIGH vulnerabilities found
      - name: Push Docker image
        uses: docker/build-push-action@v5
        with:
          push: true # Now push the image to our registry
          # ... tags and labels ...
```

**Explanation**: This job `needs` the `build` job to finish first. It downloads our built web application, logs into GitHub's Container Registry, then uses Docker to build a container image for our game. Crucially, it then uses `Trivy` to scan that image for vulnerabilities. If severe vulnerabilities are found, the pipeline stops. Only if the scan passes is the Docker image pushed to the registry.

##### Job 5: `update-k8s` (Deployment Configuration Update Station)

```yaml
# .github/workflows/ci-cd.yml (Simplified update-k8s job)
  update-k8s:
    name: Update Kubernetes Deployment
    runs-on: ubuntu-latest
    needs: [docker] # ONLY run if the 'docker' job (including scan) passed
    if: github.ref == 'refs/heads/main' && github.event_name == 'push' # Only on main branch pushes
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          token: ${{ secrets.TOKEN }} # Need special permission to push changes
          fetch-depth: 0
      - name: Update Kubernetes deployment file
        run: |
          # This command finds the image line in kubernetes/deployment.yaml
          # and replaces it with the newly built Docker image tag.
          sed -i "s|^\([[:space:]]*image:\).*|\1 ghcr.io/your-org/your-repo:sha-${GITHUB_SHA}|" kubernetes/deployment.yaml
          git diff kubernetes/deployment.yaml # Show what changed
      - name: Commit and push changes
        run: |
          git add kubernetes/deployment.yaml
          git commit -m "Update Kubernetes deployment with new image" || echo "No changes to commit"
          git push origin HEAD:main # Push the updated file back to GitHub
```

**Explanation**: This final job `needs` the `docker` job to be successful. It has an `if` condition to only run when changes are pushed directly to the `main` branch. It checks out the repository, then uses a command (`sed`) to update the `kubernetes/deployment.yaml` file to point to the *newly pushed* Docker image. Finally, it commits this change back to the `main` branch. This tells Kubernetes (our server orchestrator) that a new version of the game is ready.

#### The Full Automated Flow

Here's how these jobs form our automated factory conveyor belt when you push code to `main`:

```mermaid
sequenceDiagram
    participant Developer
    participant GitHub
    participant TestJob["Unit Testing (Job)"]
    participant LintJob["Static Analysis (Job)"]
    participant BuildJob["Build App (Job)"]
    participant DockerJob["Docker Build, Scan, Push (Job)"]
    participant UpdateK8sJob["Update Kubernetes Config (Job)"]
    participant DockerRegistry["Docker Registry"]
    participant KubernetesCluster["Kubernetes Cluster"]

    Developer->>GitHub: Push code to main branch
    GitHub->>TestJob: Trigger "test" job
    GitHub->>LintJob: Trigger "lint" job
    Note over TestJob, LintJob: These run in parallel
    TestJob-->>BuildJob: Pass (if tests successful)
    LintJob-->>BuildJob: Pass (if lint successful)
    BuildJob->>DockerJob: Pass (if build successful)
    DockerJob->>DockerRegistry: Push new Docker image
    DockerJob-->>UpdateK8sJob: Pass (if Docker image built, scanned, pushed)
    UpdateK8sJob->>GitHub: Commit updated kubernetes/deployment.yaml
    GitHub->>KubernetesCluster: (External tool) Deploys new app via config change
    Note over KubernetesCluster: New game version is live!
```

**Explanation of the flow**:
1.  **Developer pushes code**: This starts the process.
2.  **GitHub triggers jobs**: The `test` and `lint` jobs run simultaneously to quickly check code quality.
3.  **`build` waits**: The `build` job only starts after both `test` and `lint` finish successfully.
4.  **`docker` waits**: The `docker` job starts after `build` completes. It packages, scans, and pushes our game's Docker image to the `Docker Registry`.
5.  **`update-k8s` waits**: The `update-k8s` job runs last. It modifies the Kubernetes configuration file in our repository to point to the new Docker image.
6.  **Kubernetes deploys**: An external tool (or manual action) detects this change and tells the `Kubernetes Cluster` to deploy the new version of our game.

---

### Conclusion

In this chapter, we've explored the power of an **Automated CI/CD Pipeline**. We learned that it acts as an automated factory, using **GitHub Actions** and our `ci-cd.yml` instruction manual to automatically test, lint, build, package, scan for security, and update the deployment configuration of our **SecureFlow** Tic Tac Toe game every time code is changed. This ensures consistent quality, security, and efficient delivery, without requiring manual effort.

Now that we understand how our application gets built and moved along an automated pipeline, let's dive deeper into how our game is actually "packaged" into a portable format using Docker.

[Next Chapter: Docker Containerization](05_docker_containerization_.md)

---