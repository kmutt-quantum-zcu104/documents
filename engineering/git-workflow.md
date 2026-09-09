# Our Git workflow

Our team uses Git and GitHub to coordinate, manage, and track the development of both hardware (RTL) and software across the KMUTT Quantum Hardware Testbed.

Hardware projects and embedded firmware have their own unique pitfalls. Some Git commands can be destructive or cause silent merge errors in Vivado projects and makefiles. Team members come with different backgrounds in software and electrical engineering, but everyone should be able to contribute cleanly and safely.

It is therefore important that we share an understanding of how these tools are used, so we can build together without stepping on each other's toes.

## Required knowledge

### Git

[_Pro Git_](https://git-scm.com/book/en/v2), the official Git book, is free and available online.

Reading at least the first three chapters, through the end of "Git Branching," is strongly recommended.

### GitHub

Git and GitHub are not the same thing. Git is the underlying version control tool; GitHub provides collaboration workflows such as issues and pull requests.

We do not push directly to production branches. Instead, we use branches and pull requests to review each other's changes.

### Command-line interface

The rest of this guide uses the `git` command-line interface. Learning the CLI will help you understand what is actually happening under the hood, and it works identically on your laptop, a remote build server, or the ZCU104 board itself.

## Required setup

Before writing code, configure Git with your identity and standard settings to prevent confusing errors down the road.

### Commit name and email address

Set your name and email address consistently across Git and GitHub:

```sh
git config --global user.name 'Your Name'
git config --global user.email 'your_email@example.com'
```

### Essential Git settings

Run all of the commands below on your machine:
```sh
# Make the default branch name in a new repository 'main'
git config --global init.defaultBranch main

# Push the current branch to a remote branch with the same name
git config --global push.default simple

# Automatically set up tracking when pushing a new branch
git config --global push.autoSetupRemote true

# Use fast-forward or standard merge to resolve conflicts during pull
git config --global pull.ff true
git config --global pull.rebase false

# Do not let Git convert line endings behind your back
git config --global core.autocrlf false
```

### Avoiding line-ending confusion

If you work on Windows, editors and tools often default to CRLF line endings, while Linux expects LF. When line endings get mixed up, Git diffs show entire files as modified, and shell scripts on the ZCU104 will fail with bizarre syntax errors.

By default on Windows, Git sets `core.autocrlf` to `true`. This setting tries to convert LF to CRLF automatically, but in practice it frequently breaks scripts, Tcl files, and Linux source code.

* Make sure you ran `git config --global core.autocrlf false`.
* Configure your text editor (VS Code, Zed, CLion, Vim, etc.) to use **LF line endings** by default.
* We include `.editorconfig` and `.gitattributes` in our repositories to enforce LF line endings automatically.

## GitHub Flow

We follow GitHub Flow with branch protection rules on `main`.

### Create a branch

If you are a member of our organization, do not fork the repository. Clone the repository directly and create a branch:

```sh
# Ensure you are on main and up to date
git checkout main
git pull

# Create and switch to your feature branch
git checkout -b <branch-type>/<short-description>
```

We use standard branch naming prefixes:

* `feat/` for new RTL modules, drivers, or software features
* `fix/` for bug fixes or resolving hardware timing violations
* `docs/` for documentation, guides, or proposal updates
* `infra/` for build scripts, Yocto recipes, and CI configuration

### Make changes

Be deliberate about what you stage and commit. Hardware development generates many temporary files that should never touch Git.

Always inspect your changes before committing:

```sh
# Check what files have been modified or created
git status

# Stage only the files you intend to commit
git add <path-to-file>

# Review the staged changes
git status

# Commit with a clear, concise message
git commit -m '<commit message>'
```

Avoid using `git commit -a`, because it makes it too easy to accidentally stage temporary files, credentials, or massive test data.

Commit messages should be written in English and follow the [Conventional Commits](https://www.conventionalcommits.org/en) convention (for example: `feat(decoder): add lookup table module for distance 3 or docs(board): update serial console pinout`).

### Create a pull request

Keep pull requests as small and focused as possible. A PR that changes one module or adds one guide is much easier to review than a massive commit spanning half the semester.

Before creating a pull request, ensure your branch is up to date with `main`:

```sh
git checkout main
git pull
git checkout <your-branch-name>
git merge main
git push
```

Open a pull request on GitHub. Provide a brief explanation of what you changed, why, and how you verified it (such as simulation results, linting, or on-board tests).

### Address review comments

Code reviews are collaborative check-ins, not tests.

* If a reviewer asks questions or suggests improvements, feel free to discuss them either directly on GitHub or in person.
* If you discuss things in person, write a brief comment on the PR summarizing what was agreed on so the decision is recorded.
* If you are self-reviewing your own pull request, use the PR diff view to check for accidental whitespace changes, leftover debugging statements, or missing files before merging.

### Merge your pull request

Once review comments are resolved and automated checks pass, merge the pull request:

* Generally, the author of the pull request should be the one to press the merge button.
* We recommend Squash and merge for single features or documentation updates to keep the history on `main` clean and readable.

### Delete your branch

*After merging a pull request, always delete the merged branch*.

Leaving merged branches around clutters the repository and creates confusion about what is active and what is obsolete.

Once you click the "Delete branch" button on GitHub, clean up your local machine:

```sh
git checkout main
git pull
git branch -d <your-branch-name>
git remote prune origin
```

## Shared practices

### Rebasing on shared branches is discouraged

We strongly discourage rebasing branches that are shared or actively under review. Force-pushing can overwrite work done by others, confuse reviewers looking at review diffs, and erase cryptographic signatures.

Use standard `git merge main` to keep your branch up to date. Direct pushes and force-pushes to the `main` branch are strictly blocked by repository protection rules.
