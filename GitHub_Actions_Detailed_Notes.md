# GitHub Actions — Complete Beginner Notes (Detailed Edition)

> **How to use these notes:** Read each section in order. Every YAML example is fully
> annotated with comments explaining what each line does. Don't rush — the goal is
> understanding, not memorizing.

---

# PART 1 — THE FOUNDATIONS

## 1. What is GitHub Actions, really?

Imagine you hire a robot assistant for your code repository. This robot watches your
repository 24/7, and every time something happens (someone pushes code, opens a pull
request, etc.), the robot springs into action and does whatever chores you taught it:
run tests, build the app, deploy to a server, send a notification...

**That robot is GitHub Actions.**

More formally: GitHub Actions is an **automation and CI/CD platform built into GitHub**.
You don't install anything — it's already there inside every GitHub repository. You just
tell it *when* to run and *what* to do, using a simple text file (written in YAML).

### A concrete example of the robot in action

```
1. A developer writes new code on their laptop
2. They run:  git push
3. The code lands on GitHub
4. GitHub notices: "A push just happened!"
5. GitHub Actions wakes up and starts your workflow:
      - downloads your code onto a fresh machine
      - installs the project's dependencies
      - runs the test suite
      - builds the application
      - (optionally) deploys it to a server
6. The developer sees a ✅ or ❌ right on GitHub
```

If the tests fail, the developer knows *within minutes* — not days later when
someone manually tries to run everything.

### Why does this matter?

Without automation, every developer on a team has to remember to:
- run tests before merging,
- check code style,
- build the project,
- deploy carefully.

People forget. People skip steps. Automation never forgets and never skips.

---

## 2. What is CI/CD? (Explained slowly)

CI/CD is the *reason* tools like GitHub Actions exist. The letters stand for:

### CI — Continuous Integration

**What it means:** Developers merge (integrate) their code into a shared repository
*very frequently* — often several times a day — and every single time, an automated
system checks that the new code actually works.

**Break the phrase down:**
- **Continuous** = happens constantly, on every change
- **Integration** = combining everyone's code together into one shared codebase

**The old painful way (before CI):**
```
Monday    → Developer A writes feature A on their laptop for 2 weeks
Tuesday   → Developer B writes feature B on their laptop for 2 weeks
...
Friday    → They both try to merge their work at the same time
          → The code conflicts in 47 places
          → Nothing works. Everyone goes home sad.
```
This is called "integration hell."

**The CI way:**
```
Monday 9am   → Developer A pushes a small piece of feature A
             → Automated tests run automatically ✅
Monday 11am  → Developer B pushes a small piece of feature B
             → Automated tests run automatically ✅ (and it includes A's code!)
...
Friday       → Features are already merged, tested, and working
```

Every push triggers: **run tests → run linting (code-style checks) → build the app →
report the result**. If something fails, the developer who just pushed finds out
immediately, while the change is still fresh in their mind.

### CD — Continuous Delivery / Continuous Deployment

CD can mean two slightly different things — learn both:

**Continuous Delivery** — the code is automatically *prepared* for release, but a human
must press the final button to deploy:

```
code → test → build → ready for release → HUMAN APPROVES → deploy
```
Think of it like: the robot cooks the meal and plates it, but a human tastes it before
serving.

**Continuous Deployment** — *everything* is automatic, including the final deploy. No
human in the loop:

```
code → test → build → deploy (automatically)
```
The moment tests pass on the main branch, the new version goes live by itself.

> **Note:** GitHub Actions can implement either style. You choose by how you write
> your workflow (e.g., adding a manual approval step for Delivery, or not, for Deployment).

---

## 3. Git vs GitHub vs GitHub Actions — don't confuse these three

Beginners constantly mix these up. Here's the clean separation:

### Git — the version control tool
Git is a **program that runs on your computer** and tracks changes to your files over
time. It lives entirely on your machine unless you push somewhere.

```bash
git init                          # start tracking a folder
git add .                         # stage all changes
git commit -m "Initial commit"    # save a snapshot with a message
git push                          # upload your commits to GitHub
git pull                          # download the latest changes from GitHub
git branch                        # list branches
git merge                         # combine branches
```

Think of Git as a **time machine + notebook** for your code, kept on your laptop.

### GitHub — the website for hosting Git repositories
GitHub is a **website/platform** where your Git repositories can live online so other
people can access them. It provides:
- **repositories** — your project's online home
- **pull requests** — propose changes, discuss them, review code
- **issues** — track bugs and tasks
- **code review** — comment on specific lines of changes
- **releases** — package versions of your software
- **permissions** — control who can do what

Think of GitHub as **the shared office building** where everyone's copies of the
project meet.

### GitHub Actions — the automation robot
GitHub Actions **lives inside GitHub** and automates tasks around your repository.
It watches what happens on GitHub and reacts:

```
Something happens on GitHub (push, PR, schedule, manual click)
        ↓
GitHub Actions starts a workflow
        ↓
Run tests → Build → Deploy
```

**One-line summary:**
> Git tracks your code. GitHub hosts it. GitHub Actions automates work around it.

---

# PART 2 — THE CORE VOCABULARY

These six words are the foundation of everything. Understand these and 80% of GitHub
Actions is unlocked.

## 4. Workflow

A **workflow** is an automated process, defined in a YAML file, that tells GitHub
Actions: *"when X happens, do Y."*

```yaml
name: CI
```

- The `name:` line simply gives your workflow a human-readable title. It shows up in
  the GitHub web interface.
- Workflow files **must** live in a specific folder inside your repository:

```
your-repo/
└── .github/
    └── workflows/
        ├── ci.yml        ← one workflow
        └── deploy.yml    ← another workflow
```

> **Why that exact folder?** GitHub specifically looks inside `.github/workflows/` for
> YAML files. Put your workflow anywhere else and GitHub will completely ignore it —
> a very common beginner mistake.

You can have **multiple workflows** in one repository — e.g., one that runs tests on
every push, and a separate one that deploys only to production.

## 5. Job

A **job** is a collection of steps that run together, one after another, on one runner
(one machine).

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Running tests"
```

The hierarchy is:

```
WORKFLOW
   └── JOB (a named group of tasks)
         └── STEPS (the individual tasks)
```

A workflow can contain **multiple jobs**:

```yaml
jobs:
  test:
    ...
  build:
    ...
  deploy:
    ...
```

**Key fact about jobs:** by default, jobs that don't depend on each other run **in
parallel** — at the same time, on separate machines. This makes your pipelines faster.
(We'll see how to chain them with `needs:` in Part 5.)

## 6. Step

A **step** is a single task inside a job. Steps run **in order, one after another**,
and they all share the same machine (so later steps can use files created by earlier
steps *within the same job*).

```yaml
steps:
  - name: Say hello
    run: echo "Hello"

  - name: Show directory contents
    run: ls

  - name: Show Git version
    run: git --version
```

- `name:` is optional but recommended — it gives the step a readable label in the
  GitHub web UI (so instead of seeing a cryptic command, you see "Run tests").
- `run:` is the actual command to execute.

## 7. Runner

A **runner** is the machine (server) that actually executes your job. GitHub provides
managed machines you can use for free:

```yaml
runs-on: ubuntu-latest
```

This line says: *"Run this job on a GitHub-hosted machine with the latest Ubuntu Linux
installed."*

Common choices:
| Runner | What it is | When you'd use it |
|---|---|---|
| `ubuntu-latest` | A Linux machine | **Most CI/CD work** — the default choice for most projects |
| `windows-latest` | A Windows machine | Building Windows apps, .NET projects |
| `macos-latest` | A macOS machine | Building iOS/Mac apps |

**Important details about runners:**
- Every job gets a **fresh, clean machine**. Nothing is saved between workflow runs
  (unless you deliberately use caching or artifacts — covered later).
- Every job in a workflow gets its **own** machine. This is why jobs can't see each
  other's files by default.

## 8. Action (the reusable building block)

An **action** is a **reusable, pre-packaged unit of automation** — basically a piece of
workflow someone else wrote and published so you don't have to rewrite it.

```yaml
- uses: actions/checkout@v4
```

This single line performs a surprisingly complex task: it downloads (checks out) your
repository's code onto the runner so your workflow can work with it. Without this
action, you'd have to write all the git commands yourself.

**The `@v4` part** means "version 4 of this action." Actions are versioned so behavior
stays predictable.

**Where do actions come from?** Many are official (`actions/checkout`, `actions/setup-node`),
and thousands more are published on the GitHub Marketplace by the community.

## 9. YAML — the language of workflows

GitHub Actions workflows are written in **YAML** — a format that uses indentation to
show structure (like Python).

**The #1 rule of YAML: indentation is not decoration — it IS the meaning.**

This is valid — `test` belongs inside `jobs`, `run` belongs inside `steps`:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Hello"
```

This is **broken** — because of bad indentation, GitHub can't tell what belongs to what:

```yaml
jobs:
test:
runs-on: ubuntu-latest
```

**A tip that saves hours:** always use **spaces**, never tabs, in YAML. Editors can
sneak tabs in and cause mysterious errors.

---

# PART 3 — YOUR FIRST WORKFLOW, LINE BY LINE

## 10. Building your first workflow from scratch

**Step 1:** In your repository, create this file:

```
.github/workflows/hello.yml
```

**Step 2:** Paste this in:

```yaml
name: Hello World

on:
  push:

jobs:
  hello:
    runs-on: ubuntu-latest

    steps:
      - name: Say hello
        run: echo "Hello boss!"
```

**Step 3:** Commit the file and push it to GitHub.

**Step 4:** Go to your repo → click the **"Actions"** tab at the top. You'll see your
workflow running! Click it to watch each step execute in real time.

## 11. Reading the workflow — every line explained

```yaml
name: Hello World
```
→ Gives the workflow a title. Purely cosmetic, but helpful when you have several.

```yaml
on:
  push:
```
→ **The trigger.** This says: "Run this workflow every time anyone pushes code to this
repository." (`on:` is a special YAML key in GitHub Actions — note it's literally the
word "on".)

```yaml
jobs:
```
→ Begins the section where you define the jobs this workflow contains.

```yaml
  hello:
```
→ **The job ID** — a short name you choose. It's an identifier, used if other jobs need
to reference this one (with `needs:`).

```yaml
    runs-on: ubuntu-latest
```
→ The runner: run this job on a fresh GitHub-hosted Ubuntu Linux machine.

```yaml
    steps:
```
→ Begins the list of tasks for this job.

```yaml
      - name: Say hello
        run: echo "Hello boss!"
```
→ A single step: give it a friendly name, and define the shell command to execute.
`echo` just prints text to the log. So this step literally prints "Hello boss!" —
and that's it. You just ran your first workflow. 🎉

---

# PART 4 — TRIGGERS: WHEN DOES THE WORKFLOW RUN?

The `on:` section is how you control **when** your automation fires.

## 12. Push trigger

```yaml
on:
  push:
```
Runs on **every push** to **any branch**. Simple and common.

## 13. Pull request trigger

```yaml
on:
  pull_request:
```
Runs when pull request activity happens — opened, updated with new commits, reopened.
This is the backbone of "test every PR before merging."

## 14. Multiple triggers

```yaml
on:
  push:
  pull_request:
```
A workflow can listen for several events. Either one will start it.

## 15. Restricting to specific branches

You often don't want to run heavy pipelines on every experimental branch:

```yaml
on:
  push:
    branches:
      - main
```
→ Now the workflow only runs when someone pushes to `main`.

Multiple branches:

```yaml
on:
  push:
    branches:
      - main
      - develop
```

## 16. PRs targeting specific branches

```yaml
on:
  pull_request:
    branches:
      - main
```
→ Runs when a pull request is opened/updated **whose target (destination) branch is
`main`**. Note the subtle difference from the push trigger: this is about which branch
the PR wants to merge *into*.

## 17. Manual trigger (run it yourself)

```yaml
on:
  workflow_dispatch:
```
This adds a "Run workflow" button in the GitHub web UI (Actions tab → pick workflow →
"Run workflow"). Great for deployments or one-off tasks.

You can combine manual with automatic:

```yaml
on:
  push:
    branches:
      - main
  workflow_dispatch:
```
→ Runs automatically on pushes to main, OR whenever you click the button.

## 18. Scheduled trigger (run on a timer)

```yaml
on:
  schedule:
    - cron: "0 0 * * *"
```
→ Runs on a schedule using **cron syntax** — a compact time format used across all of
Unix/Linux:

```
┌───────── minute (0-59)
│ ┌─────── hour (0-23)
│ │ ┌───── day of month (1-31)
│ │ │ ┌─── month (1-12)
│ │ │ │ ┌─ day of week (0-6, Sunday=0)
* * * * *
```

So `"0 0 * * *"` = "at minute 0 of hour 0, every day, every month, every weekday" =
**midnight UTC every day**.

More cron examples:
| Cron | Meaning |
|---|---|
| `0 0 * * *` | Every day at midnight UTC |
| `0 6 * * 1` | Every Monday at 6am UTC |
| `*/15 * * * *` | Every 15 minutes |

> ⚠️ **Warning:** Scheduled workflows run in **UTC** time. Convert to your timezone
> when picking a schedule. Also note GitHub delays scheduled runs under heavy load,
> so don't use them for tasks requiring exact timing.

---

# PART 5 — THE ESSENTIAL WORKING PARTS

## 19. Checkout — the step everyone needs

Here's something surprising: **when a runner starts, your repository's files are NOT
there.** The machine is fresh — it has an operating system and some standard tools,
but not your code.

So for almost every real workflow, the first step is:

```yaml
- name: Checkout repository
  uses: actions/checkout@v4
```

This action clones your repo onto the runner. After this step, all your files are
available to later steps.

```yaml
name: CI
on:
  push:
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: List files
        run: ls -la
```

The `ls -la` step will now show your actual project files. Without the checkout step,
it would show an empty directory.

## 20. `run` vs `uses` — the single most important distinction

**`run`** = execute a **shell command** you wrote yourself.

```yaml
- name: Show Git version
  run: git --version
```

This runs `git --version` on the runner's command line, exactly as if you typed it.

**`uses`** = use a **pre-built action** (someone else's packaged automation).

```yaml
- uses: actions/checkout@v4
```

This invokes the official checkout action (which internally runs its own commands).

**Memory hook:**
> `run` → *you* write the command.
> `uses` → *someone else* wrote the commands; you're borrowing them.

## 21. Running multiple commands in one step

If you have several related commands, use the pipe `|` to write a multi-line script:

```yaml
- name: Information
  run: |
    echo "Current directory:"
    pwd

    echo "Files:"
    ls -la

    echo "Git version:"
    git --version
```

All of these commands run **in the same step, in the same shell session**. Why does
that matter? Because a later command can depend on an earlier one — e.g., `cd` into a
folder, then run something inside it.

---

# PART 6 — CONFIGURATION: VARIABLES & SECRETS

## 22. Environment variables

**Environment variables** are named values you can use to configure commands without
hard-coding values inside commands.

```yaml
env:
  APP_NAME: my-app

steps:
  - name: Print application name
    run: echo "$APP_NAME"
```
→ The shell variable `$APP_NAME` gets replaced with `my-app` when the command runs.

**Variables can be defined at three levels**, from broadest to narrowest:

**Level 1 — Workflow level** (available to ALL jobs and steps):

```yaml
name: CI
env:
  APP_NAME: my-app      # ← visible everywhere in this workflow
on:
  push:
jobs:
  ...
```

**Level 2 — Job level** (available only within that job):

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    env:
      APP_NAME: my-app   # ← only this job's steps can see it
    steps:
      - run: echo "$APP_NAME"
```

**Level 3 — Step level** (available only in that one step):

```yaml
steps:
  - name: Example
    env:
      MESSAGE: "Hello"   # ← only this step can see it
    run: echo "$MESSAGE"
```

If the same variable is defined at multiple levels, the **most specific (narrowest)
level wins** — step level beats job level, job level beats workflow level.

## 23. Secrets — never put passwords in your code

**The rule:** never write sensitive values (passwords, API keys, tokens) directly in a
workflow file. Why? Because workflow files are stored in your repository — anyone with
read access to the repo can see them. And if the repo is ever public, your secrets are
public forever.

**Bad (never do this):**

```yaml
env:
  PASSWORD: mypassword      # ❌ anyone can read this
  API_KEY: abc123           # ❌ this is now compromised
```

**The solution: GitHub Secrets.** These are encrypted values stored in your repository
(or organization/environment) settings — visible to no one. The workflow can *use* them
at runtime, but no one can read them back out.

**Typical secrets:** API keys, cloud credentials (AWS keys), deployment tokens,
database passwords, private access tokens.

**How to use a secret in a workflow:**

```yaml
steps:
  - name: Deploy
    env:
      API_KEY: ${{ secrets.API_KEY }}
    run: ./deploy.sh
```

`${{ secrets.API_KEY }}` is the syntax for "insert the value of the secret named
`API_KEY` here, at runtime." The value never appears in any file.

**How to create a secret:** GitHub repo → **Settings** → **Secrets and variables** →
**Actions** → **New repository secret**. Give it the exact name you reference in the
workflow (names are case-sensitive).

## 24. Variables vs Secrets — when to use which

| | Variables | Secrets |
|---|---|---|
| **For** | Normal, non-sensitive configuration | Sensitive values |
| **Examples** | `APP_ENV=production`, `REGION=us-east-1`, `PORT=3000` | `API_KEY`, `DATABASE_PASSWORD`, `DEPLOY_TOKEN` |
| **Storage** | Plain text in repo settings | Encrypted in repo settings |
| **Syntax** | `${{ vars.MY_VAR }}` | `${{ secrets.MY_SECRET }}` |

**Rule of thumb:** If leaking the value would hurt you, it's a secret.

---

# PART 7 — EXPRESSIONS, CONTEXTS & CONDITIONS

## 25. Expressions — the `${{ }}` syntax

GitHub Actions has a mini expression language. Anywhere you write `${{ ... }}`, GitHub
evaluates the expression and replaces it with the result.

```yaml
run: echo "${{ github.ref }}"
```
→ At runtime, this becomes something like `echo "refs/heads/main"`.

## 26. The `github` context — information GitHub gives you for free

A **context** is a bundle of information about the current workflow run, provided
automatically. The most useful one is `github`:

| Expression | What it gives you |
|---|---|
| `${{ github.repository }}` | The repo name, e.g. `myuser/my-project` |
| `${{ github.ref }}` | The Git ref, e.g. `refs/heads/main` |
| `${{ github.actor }}` | The username of whoever triggered the run |
| `${{ github.sha }}` | The exact commit hash (SHA) being built |

**Why this is powerful:** your workflow can *know things about itself* — which branch
it ran on, who triggered it, which commit. You'll use these constantly in real pipelines
(e.g., tagging a Docker image with `${{ github.sha }}` so you know exactly which commit
is deployed).

## 27. Conditions — deciding whether something runs

You can attach an `if:` to a job or step. It only runs if the condition is true.

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - run: echo "Deploying application..."
```

→ The `deploy` job runs **only** if the workflow was triggered from the `main` branch.

Step-level condition:

```yaml
- name: Deploy
  if: github.ref == 'refs/heads/main'
  run: ./deploy.sh
```

→ Only this *step* is conditional — the rest of the job runs normally.

**Why useful:** you might run tests on every branch but only deploy from `main`.
Conditions let one workflow handle both cases.

---

# PART 8 — CHAINING JOBS TOGETHER

## 28. `needs:` — making jobs run in sequence

Remember: by default, jobs run **in parallel**. But real pipelines usually need order:

```
first run the tests → only if tests pass, build → only if build succeeds, deploy
```

You express this with `needs:`:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Testing"

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building"

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying"
```

**What `needs:` means:**
- `build` **waits** for `test` to finish successfully, then starts.
- `deploy` waits for `build`.
- **If `test` fails, `build` never runs** — and neither does `deploy`. The pipeline
  stops at the first failure. This is exactly the safety you want: you never deploy
  untested code.

## 29. Parallel jobs — when order doesn't matter

Some jobs genuinely don't depend on each other. Let them run simultaneously:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Testing"

  lint:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Linting"
```

Both spin up on separate machines at the same time. Total wall-clock time ≈ the slower
of the two, not the sum. On bigger pipelines, parallelism is a major speedup.

---

# PART 9 — REAL CI EXAMPLES, STEP BY STEP

## 30. Installing a language runtime

Runners come with many tools pre-installed, but for a specific language version, use
the official setup actions:

```yaml
- name: Setup Node
  uses: actions/setup-node@v4
  with:
    node-version: 20
```

The `with:` block passes **inputs** (parameters) to the action — here, "install Node.js
version 20."

Then the standard rhythm for Node projects:

```yaml
- name: Install dependencies
  run: npm ci          # installs packages from package-lock.json, exactly

- name: Run tests
  run: npm test
```

> 💡 `npm ci` vs `npm install`: `npm ci` installs the *exact* versions from the lock
> file — faster and more reproducible. It's the right choice for CI.

## 31. Complete Node.js CI pipeline (fully annotated)

```yaml
name: Node CI

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      # 1. Get the code onto the runner
      - name: Checkout repository
        uses: actions/checkout@v4

      # 2. Install Node.js 20
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      # 3. Download project dependencies
      - name: Install dependencies
        run: npm ci

      # 4. Run the test suite
      - name: Run tests
        run: npm test

      # 5. Build the production bundle
      - name: Build
        run: npm run build
```

**The pipeline in pictures:**

```
Push or PR to main
        ↓
   Checkout code
        ↓
  Install Node 20
        ↓
  Install dependencies (npm ci)
        ↓
    Run tests  ← if any test fails, everything stops here ❌
        ↓
   Build app   ← only reached if tests pass ✅
```

Any step failing stops the job and marks the whole run as failed.

## 32. Complete Python CI pipeline

```yaml
name: Python CI

on:
  push:
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      # 1. Get the code
      - uses: actions/checkout@v4

      # 2. Install Python 3.12
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      # 3. Upgrade pip, then install project dependencies
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      # 4. Run tests with pytest
      - name: Run tests
        run: pytest
```

---

# PART 10 — POWER FEATURES

## 33. Matrix builds — test many versions at once

**Problem:** you want to verify your app works on Python 3.10, 3.11, *and* 3.12.
Writing three identical jobs would be tedious and error-prone.

**Solution:** a **matrix** generates a job for each combination automatically:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        python-version:
          - "3.10"
          - "3.11"
          - "3.12"

    steps:
      - uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run tests
        run: pytest
```

This single job definition creates **three parallel jobs** — one per Python version —
using `${{ matrix.python-version }}` to fill in the current version in each.

Conceptually:

```
            ┌── Python 3.10 → install → pytest
start ──────┼── Python 3.11 → install → pytest
            └── Python 3.12 → install → pytest
```

Matrices can span multiple dimensions (e.g., OS × Node version) and are a cornerstone
of professional CI.

## 34. Artifacts — passing files between jobs (and keeping outputs)

**The problem:** each job runs on its **own separate machine**. If your `build` job
produces a `dist/` folder, your `deploy` job can't see it — it's on a different machine!

**The solution: artifacts.** An artifact is a file/folder uploaded from one job and
downloadable by another job (or by you, from the workflow run page).

**Uploading:**

```yaml
- name: Upload artifact
  uses: actions/upload-artifact@v4
  with:
    name: build-files      # a label for this artifact
    path: dist/            # the folder to upload
```

**Downloading (in another job):**

```yaml
- name: Download artifact
  uses: actions/download-artifact@v4
  with:
    name: build-files      # must match the upload name
```

**The pattern:**

```
┌── build job ──→ creates dist/ ──→ upload artifact ──┐
│                                                     ↓
└──────────────── deploy job ←── download artifact ←──┘
```

Artifacts are also useful for keeping test reports, logs, or coverage results after
a run finishes — download them from the run's page for inspection.

## 35. Caching — making workflows faster

**The problem:** `npm ci` / `pip install` download packages from the internet every run.
If dependencies rarely change, you're re-downloading the same megabytes every time,
slowing every run.

**The solution: caching** — save downloaded packages after a run; restore them on the
next run. Some setup actions have caching built in:

```yaml
- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    node-version: 20
    cache: npm          # ← automatically caches npm's download cache
```

This single line can cut minutes off repeated runs. (There is also a general-purpose
`actions/cache` action for anything the setup actions don't cover.)

## 36. Permissions — the principle of least privilege

Workflows get a token (`GITHUB_TOKEN`) they can use to talk to your repo (push commits,
create releases, comment on PRs...). **By default, workflows get fairly broad access.**
Best practice: grant only what's needed:

```yaml
permissions:
  contents: read
```
→ "This workflow may only *read* repository contents. Nothing else."

Only elevate permissions when a specific task requires it (e.g., `contents: write` to
push a commit). This limits the damage if a workflow or a third-party action is ever
compromised.

## 37. Pull Request CI — the most common real-world use

```yaml
name: Pull Request CI

on:
  pull_request:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test
```

**What happens in real life:**

1. Developer creates a pull request → workflow triggers automatically.
2. GitHub shows a checks section on the PR: *"test — in progress..."*
3. Tests pass → ✅ green check → the team can merge confidently.
4. Tests fail → ❌ red X → the developer fixes the code *before* merging.

This is the core safety loop of modern software development.

## 38. Branch protection — enforcing the safety loop

Testing PRs is useless if someone can just ignore failing tests and merge anyway.
**Branch protection rules** fix that:

Repo → **Settings** → **Branches** → **Add branch protection rule** for `main` →
enable **"Require status checks to pass before merging."**

Now GitHub **physically blocks the merge button** until your workflow passes:

```
Developer opens PR
        ↓
GitHub Actions runs tests
        ↓
   Tests pass?  ──NO──→ ❌ Merge button disabled → fix code and push again
      ↓ YES
   ✅ Merge allowed
```

## 39. Docker with GitHub Actions

GitHub-hosted runners have Docker pre-installed, so building images is trivial:

```yaml
- name: Build Docker image
  run: docker build -t my-app .

- name: Run Docker container (smoke test)
  run: docker run my-app
```

A production pipeline would then log in to a container registry and push the image:

```
code → test → build → docker build → docker push (to Docker Hub / ECR / GHCR)
                                          ↓
                              production server pulls the image
```

## 40. Deployment — where the pipeline ends

GitHub Actions can deploy to almost anything:
- **Cloud platforms:** AWS, Azure, Google Cloud
- **Virtual machines:** your own server via SSH
- **Kubernetes:** container orchestration
- **Static hosting:** GitHub Pages, Netlify, Vercel
- **Container platforms:** after pushing an image

A simplified end-to-end pipeline:

```
Developer pushes
     ↓
GitHub
     ↓
Tests run
     ↓
App builds
     ↓
Docker image built
     ↓
Image pushed to registry
     ↓
Production server pulls & runs the new image
```

Deployment jobs typically use secrets (server credentials, cloud keys) — never hard-coded.

## 41. Environments — organizing deployment stages

GitHub **environments** represent deployment targets: `development`, `staging`,
`production`. You can attach **secrets and protection rules** to each:

```yaml
deploy:
  runs-on: ubuntu-latest
  environment: production
  steps:
    - run: ./deploy.sh
```

**Why this is powerful:**
- Secrets can be scoped to an environment (the production database password is only
  available to jobs deploying to production).
- Protection rules can **require a manual approval** before a job targeting production
  runs — implementing Continuous Delivery's "human approval" step.
- You get a deployment history per environment.

## 42. A complete production deployment pattern

```yaml
name: Production Deployment

on:
  push:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: build-files
          path: dist/

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build-files
      - run: echo "Deploying application..."
```

**How it flows:**

```
Push to main
    │
    ▼
┌─────── test job ────────┐
│ checkout → npm ci       │
│ → npm test              │
└────── on success ───────┘
    │
    ▼
┌─────── build job ───────┐
│ checkout → npm ci       │
│ → npm run build         │
│ → upload dist/ artifact │
└────── on success ───────┘
    │
    ▼
┌─────── deploy job ──────┐
│ download artifact       │
│ → deploy (production    │
│   environment rules     │
│   + secrets apply here) │
└─────────────────────────┘
```

Note how each job runs on its **own machine** — the artifact upload/download is what
bridges `build` → `deploy`.

---

# PART 11 — ADVANCED & ENTERPRISE FEATURES

## 43. Self-hosted runners

Everything so far used **GitHub-hosted runners** (GitHub's machines, free minutes
included). A **self-hosted runner** is a machine **you** own and manage — your server,
your laptop, a VM in your company network — that you register with GitHub to execute
your workflows.

```
Your server
     ↓
Self-hosted GitHub Actions runner (a small agent program)
     ↓
Executes your workflows on YOUR hardware/network
```

**Why use one?**
- **Custom hardware:** GPU builds, specific CPU architectures
- **Private network access:** reach internal databases/services GitHub's cloud can't
- **Specialized software:** licensed tools, huge pre-installed environments
- **Internal infrastructure:** deploy directly to machines in your datacenter

**The trade-offs:** you are responsible for security, patching, and maintenance of that
machine. A compromised self-hosted runner that handles public PRs is a serious risk —
GitHub warns against using self-hosted runners with public repositories.

## 44. Reusable workflows

Large organizations have dozens of repos that all need the *same* CI. Instead of
copy-pasting the workflow into every repo (which then drifts out of sync), you write
the workflow **once** in a central repository and call it from others:

```
Repository A ──┐
Repository B ──┼──→ "Call the shared CI workflow" → runs centrally-defined pipeline
Repository C ──┘
```

Any improvement to the shared workflow automatically benefits every repo using it.

## 45. Composite actions

**Reusable workflows** share whole workflows; **composite actions** share *step
sequences*. Suppose every project does:

```
install dependencies → configure environment → run custom setup
```

You can bundle those steps into your own action and then use it in one line:

```yaml
- uses: my-org/my-setup-action@v1
```

Think of it as writing your own `uses:` building blocks.

## 46. Workflow commands — talking to the runner

GitHub Actions recognizes special commands printed to the log. A handy one: **log
grouping**, which collapses noisy output:

```yaml
- name: Build with grouped logs
  run: |
    echo "::group::Build"
    npm run build
    echo "::endgroup::"
```

Everything between `::group::` and `::endgroup::` appears as a collapsible section in
the web UI — much easier to navigate big logs.

## 47. Job outputs — passing data between jobs

Jobs can pass small pieces of text to later jobs via **outputs**:

**Producing job:**

```yaml
jobs:
  generate:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.version.outputs.version }}
    steps:
      - id: version                                  # step id, referenced above
        run: echo "version=1.0.0" >> "$GITHUB_OUTPUT"
```

The special `$GITHUB_OUTPUT` file is how a step reports a named value upward.

**Consuming job:**

```yaml
  release:
    needs: generate
    runs-on: ubuntu-latest
    steps:
      - run: echo "Releasing version ${{ needs.generate.outputs.version }}"
```

**When to use:** e.g., job 1 computes a version number or a deployment target, job 2
uses it. (For files/folders, use artifacts instead.)

## 48. Failure handling — `if: failure()` and `if: always()`

By default, if a step fails, the job stops immediately. Sometimes you want cleanup or
diagnostics **even on failure**:

```yaml
- name: Upload logs
  if: always()
  uses: actions/upload-artifact@v4
  with:
    name: logs
    path: logs/
```

- `if: always()` → run this step no matter what happened before
- `if: failure()` → run this step *only if* a previous step failed

**Classic use case:** upload test logs and screenshots after a failed test run, so you
can debug what happened.

## 49. Timeouts — never let a job hang forever

A stuck command (waiting for user input, a hung network call) could burn runner
minutes for hours:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 10
```

If the job exceeds 10 minutes, GitHub kills it. Set timeouts on any job that could
plausibly hang.

## 50. Concurrency — preventing overlapping runs

Imagine two pushes to main in quick succession — two deployments would overlap. For
production, that's dangerous. Concurrency controls fix it:

```yaml
concurrency:
  group: production-deploy
  cancel-in-progress: false
```

- `group:` names the "slot" — only one run per group at a time; later runs wait.
- Setting `cancel-in-progress: true` instead **cancels** the older run when a newer
  one arrives (great for CI runs where only the latest matters).

---

# PART 12 — SECURITY (TAKE THIS SERIOUSLY)

## 51. Security best practices

### 1. Never hard-code secrets
Already covered — always use `${{ secrets.* }}`. Even in private repos.

### 2. Don't blindly trust third-party actions
An action is code that runs with your workflow's permissions. A malicious (or
compromised) action could steal your secrets. Mitigations:
- Prefer **official `actions/*`** actions when available
- Check who maintains a third-party action; look at its source
- Pin to a full commit SHA for critical workflows (strongest) or at least a version tag

### 3. Pin actions appropriately
`uses: some/action@v1` follows a moving tag — the action's author could change what
`v1` points to. Security-sensitive environments pin to an exact commit:

```yaml
- uses: actions/checkout@08eba0b213ba2ff44d3c20e12b5e2e1b51f03a35  # full SHA
```

### 4. Use minimal permissions
Default to `permissions: contents: read` and grant more only where needed.

### 5. Be careful with pull requests from forks
Anyone on the internet can fork your public repo and open a PR — and that PR's
workflow runs **their code**. If your workflow hands secrets or write permissions to
PR-triggered runs, an attacker can exfiltrate secrets.

GitHub's default protections help (secrets aren't passed to fork PR runs), but don't
weaken those protections casually (e.g., with `pull_request_target`) unless you
deeply understand the implications.

---

# PART 13 — FIXING THINGS WHEN THEY BREAK

## 52. Debugging a failed workflow — a calm, systematic method

When a workflow fails, **don't panic and don't randomly change things.** Follow this:

**Step 1 — Open the workflow run.**
GitHub repo → **Actions** tab → click the workflow name → click the failed run.

**Step 2 — Find the failed job.**
The run shows each job with ✅ or ❌. Click the ❌ one (e.g., `test`).

**Step 3 — Find the failed step.**
Inside the job, each step is expandable. The failed step is marked ❌. Open it.

**Step 4 — Read the actual error.**
Scroll to the bottom of the step's log — the error message is there. Examples:
- `npm ERR!` → something wrong with dependencies/test
- `pytest failed` → a test failed
- `command not found` → the tool isn't installed / wrong path

**Step 5 — Reproduce locally.**
Run the exact same command on your own machine:
```bash
npm test
```
- **Fails locally too?** → It's an application bug. Fix the code.
- **Passes locally but fails in Actions?** → The environments differ. Investigate:
  - operating system (works on Windows, fails on Ubuntu?)
  - runtime version (Node 18 locally vs 20 in Actions?)
  - environment variables / secrets (missing in Actions?)
  - dependencies (lock file committed?)
  - permissions or working directory
  - file paths (case sensitivity differs between Windows and Linux!)

## 53. Common beginner mistakes — and their fixes

### Mistake 1: Wrong YAML indentation
**Symptom:** Workflow doesn't appear in the Actions tab, or "workflow file issue" errors.
**Fix:** Indentation defines structure. Use spaces only. Compare against a known-good
example. A YAML validator (or your editor's YAML linter) catches most of these.

### Mistake 2: Forgetting checkout
**Symptom:** `No such file or directory`, or your files/commands mysteriously missing.
**Fix:** If your steps need repository files, start with:
```yaml
- uses: actions/checkout@v4
```

### Mistake 3: Wrong runtime version
**Symptom:** `SyntaxError` or "feature not supported" in Actions but works locally.
**Fix:** Match your local version: check `node --version` / `python --version` and set
the same in the setup action's `with:` block.

### Mistake 4: Secrets not available
**Symptom:** Empty/undefined secret values.
**Fix checklist:**
- Secret name spelled exactly right (case-sensitive!)
- Secret created at the right scope (repo vs environment — if the job uses
  `environment: production`, the secret must exist in *that environment*)
- Workflow event allows secret access (fork PRs get no secrets by design)
- For environment secrets: job must specify the environment

### Mistake 5: Wrong branch condition
**Symptom:** A step/job with `if: github.ref == 'refs/heads/main'` never runs.
**Fix:** The condition compares against the branch the workflow **ran on**, not where
the file lives. PRs have refs like `refs/pull/123/merge`, not a branch name — for PRs,
check `github.base_ref` or `github.event_name` instead.

### Mistake 6: Assuming jobs share files
**Symptom:** "file not found" in a later job that built/downloaded something earlier.
**Fix:** Each job = separate machine. Pass files with **artifacts** (or merge the jobs).

---

# PART 14 — PUTTING IT ALL TOGETHER

## 54. A practical, realistic CI pipeline

```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Test
        run: npm test

      - name: Build
        run: npm run build
```

**The flow:**

```
Push or PR (to main or develop)
        ↓
    Checkout
        ↓
  Setup Node (+ cache for speed)
        ↓
Install dependencies
        ↓
   Lint (code style)  ← fails fast on style issues
        ↓
   Test  ← fails here if anything is broken
        ↓
   Build  ← verify the app actually compiles
```

## 55. How the files sit in a real project

```
my-project/
│
├── .github/
│   └── workflows/
│       ├── ci.yml          ← testing/CI pipeline
│       └── deploy.yml      ← deployment pipeline
│
├── src/                    ← your source code
├── tests/                  ← your tests
├── Dockerfile              ← container definition
├── package.json            ← Node dependencies/scripts
└── README.md
```

## 56. The mental model — keep this picture in your head

```
WORKFLOW  (one YAML file = one automated process)
    │
    ├── Trigger (when it runs: push, PR, schedule, manual)
    │
    └── JOB  (a group of steps on one machine)
          │
          ├── Runner (ubuntu / windows / macos)
          │
          └── STEPS  (run in order)
                ├── Action  (uses: someone else's packaged automation)
                ├── Command (run: your shell command)
                ├── Action
                └── Command
```

A concrete walk-through:

```
Workflow "CI"
   ↓ triggered by push to main
Job "test"
   ↓ on an Ubuntu runner
Step 1: checkout  (uses)
   ↓
Step 2: setup Node (uses)
   ↓
Step 3: npm ci    (run)
   ↓
Step 4: npm test  (run)
   ↓
Step 5: npm run build (run)
```

When this structure feels natural, everything else in GitHub Actions is just variations
on it.

---

# PART 15 — QUICK REFERENCE

## 57. Syntax cheat sheet

```yaml
name: CI                                  # workflow title
on:                                       # triggers
  push:
    branches: [main]
jobs:
  test:                                   # job id
    runs-on: ubuntu-latest                # runner
    needs: another-job                    # run after that job
    if: github.ref == 'refs/heads/main'   # condition
    timeout-minutes: 10
    environment: production
    env:
      NODE_ENV: production                # job-level variable
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4         # use an action
      - uses: actions/setup-node@v4
        with:
          node-version: 20                # action inputs
      - name: Run tests                   # named step
        run: npm test                     # shell command
      - run: |
          echo "line one"
          echo "line two"                 # multi-line command
```

Common references:
- Secret: `${{ secrets.API_KEY }}`
- Variable: `${{ vars.MY_VAR }}`
- Contexts: `${{ github.repository }}`, `${{ github.sha }}`, `${{ github.ref }}`
- Matrix value: `${{ matrix.python-version }}`
- Job output: `${{ needs.job-id.outputs.output-name }}`
- Manual trigger: `on: workflow_dispatch:`

## 58. The 18 concepts to master first

Don't memorize everything. Master these and the rest follows:

1. **Workflow** — the automated process (YAML file)
2. **Trigger** — when it runs (`on:`)
3. **Job** — a group of steps
4. **Runner** — the machine that executes
5. **Step** — one task
6. **`run`** — execute a command
7. **`uses`** — use an action
8. **`actions/checkout`** — get your code onto the runner
9. **Environment variables** — configuration
10. **Secrets** — safe credential storage
11. **Expressions `${{ }}`** — dynamic values
12. **`if`** — conditions
13. **`needs`** — job ordering
14. **Artifacts** — files between jobs
15. **Matrix** — testing many configurations
16. **Environments** — deployment stages + rules
17. **CI/CD** — the whole point of all of this
18. **Deployment** — the final destination

---

# PART 16 — HOW TO LEARN THIS (A ROADMAP)

## 59. The staged learning path

**Stage 1 — Git fundamentals** (nothing in Actions makes sense without Git)
```
git clone / status / add / commit / push / pull / branch / switch / merge
```

**Stage 2 — GitHub basics**
```
repository → branch → commit → pull request → merge → issues
```

**Stage 3 — GitHub Actions fundamentals**
```
workflow → trigger → job → runner → step → action
```

**Stage 4 — Write workflows by hand**
```
hello.yml  →  test.yml  →  build.yml
```

**Stage 5 — Build real CI pipelines**
```
checkout → install dependencies → lint → test → build
```

**Stage 6 — Configure safely**
```
secrets → environment variables → contexts → permissions
```

**Stage 7 — Advanced features**
```
matrix → artifacts → caching → job outputs → conditions
→ environments → concurrency → reusable workflows
```

**Stage 8 — Deployment**
```
GitHub → test → build → Docker → registry → server
```

## 60. Hands-on projects (learning by doing is non-negotiable)

**Project 1 — Hello Actions.** A workflow that prints `Hello GitHub Actions!` on push.
*You learn: file location, trigger, job, step, run.*

**Project 2 — Node CI.** Create a small Node app. Workflow: checkout → setup Node →
`npm ci` → `npm test` → `npm run build`.
*You learn: the standard CI rhythm.*

**Project 3 — Python CI.** Same idea: checkout → setup Python → install → `pytest`.
*You learn: actions are the same pattern across languages.*

**Project 4 — Matrix testing.** Test your app on Node 18, 20, and 22 in parallel.
*You learn: strategy/matrix.*

**Project 5 — Docker.** Write a Dockerfile; Actions builds the image and runs it as a
smoke test.
*You learn: Docker in CI.*

**Project 6 — Full CI/CD.** PR opens → tests run → merge only if green → build →
deploy on push to main.
*You learn: needs, artifacts, environments, secrets, branch protection — the real deal.*

---

### Final thought

GitHub Actions is not hard — it's just a set of simple ideas stacked neatly:
**a workflow listens for an event and runs jobs, which run steps on machines.**
Master the vocabulary, build the projects above, and within a few weeks this will feel
like second nature. Good luck! 🚀
