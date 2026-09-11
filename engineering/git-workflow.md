# Our Git workflow

Our team uses Git and GitHub to coordinate, manage, and track the development of both hardware (RTL) and software across the KMUTT Quantum Hardware Testbed.

Hardware projects and embedded firmware generate files that are easy to commit accidentally, and source changes can conflict when people work in parallel. Team members come with different backgrounds in software and electrical engineering, but everyone should be able to contribute cleanly and safely.

It is therefore important that we share an understanding of how these tools are used, so we can build together without stepping on each other's toes.

This guide recommends a lightweight workflow. It does not establish organization-wide branch naming, merge policies, or GitHub protection settings; confirm repository-specific requirements before relying on them.

## Required knowledge

### Git

[_Pro Git_](https://git-scm.com/book/en/v2), the official Git book, is free and available online.

Reading at least the first three chapters, through the end of "Git Branching," is strongly recommended.

### GitHub

Git and GitHub are not the same thing. Git is the underlying version control tool; GitHub provides collaboration workflows such as issues and pull requests.

[GitHub Flow](https://docs.github.com/en/get-started/using-github/github-flow) introduces the branch-and-pull-request workflow used in the examples below.

### Command-line interface

The rest of this guide uses the `git` command-line interface. Basic familiarity with the shell will help you follow it; these commands can also help explain what a graphical Git client is doing.

## Required setup

Before committing, configure Git with your identity and review the settings below. Examples using `--global` affect all repositories for your account on that machine. If you need a repository-specific setting instead, use `--local` from inside that repository.

### Commit name and email address

Set your name and email address consistently across Git and GitHub:

```sh
git config --global user.name 'Your Name'
git config --global user.email 'your_email@example.com'
```

### Suggested Git settings

These settings support the examples below. Review them before changing an existing setup:

```sh
# Make the default branch name in a new repository 'main'
git config --global init.defaultBranch main

# Push the current branch to a remote branch with the same name
git config --global push.default simple

# Automatically set up tracking when pushing a new branch
git config --global push.autoSetupRemote true

# Use fast-forward when possible, otherwise merge during a plain git pull
git config --global pull.ff true
git config --global pull.rebase false

# Do not let Git convert line endings behind your back
git config --global core.autocrlf false
```

### Avoiding line-ending confusion

If you work on Windows, editors and tools often default to CRLF line endings, while Linux expects LF. When line endings get mixed up, Git diffs show entire files as modified, and shell scripts on the ZCU104 will fail with bizarre syntax errors.

Some Git for Windows installations enable `core.autocrlf`, which converts line endings on checkout and commit. CRLF in a working copy can cause problems for scripts intended to run on Linux.

* For repositories that preserve line endings without conversion, use `core.autocrlf false` at the appropriate configuration scope.
* Configure your text editor (VS Code, Zed, CLion, Vim, etc.) to use **LF line endings** for Linux-oriented files.
* This repository's `.editorconfig` requests LF from supporting editors. Its `.gitattributes` uses `* -text` to disable Git's text conversion; it does not enforce LF or repair existing CRLF files. Check the configuration of other repositories rather than assuming it is identical.

## GitHub Flow

Prefer a focused branch and pull request for collaborative changes. The examples assume the shared repository is named `origin` locally and its main branch is `main`. Replace quoted placeholders such as `'<your-branch-name>'` with your own values.

Start with `git status`. Commit your current work on the appropriate branch or safely set it aside before switching branches; do not discard changes just to follow an example.

### Create a branch

If you have write access to the repository, you can create a branch in it. Otherwise, use a fork and follow [GitHub's fork workflow](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks); its remotes differ from these examples. Organization membership alone does not guarantee write access.

```sh
# Ensure you are on main and up to date
git checkout main
git pull --ff-only

# Create and switch to your feature branch
git checkout -b '<branch-type>/<short-description>'
```

`--ff-only` stops rather than creating a merge if your local `main` has diverged. If it fails, inspect the history and ask for help if needed; do not reset away local commits to make it succeed.

Descriptive branch names help others understand the work. Possible prefixes include:

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
git diff

# Stage only the files you intend to commit
git add '<path-to-file>'

# Review the contents selected for this commit
git diff --cached
git status

# Commit with a clear, concise message
git commit -m '<commit message>'
```

`git diff` does not show untracked files, so inspect new files as well. After staging, `git diff --cached` shows the selected content; `git status` shows which files are staged or still unstaged.

Prefer explicit staging over `git commit -a`: `-a` stages modifications and deletions of all tracked files, which may include unrelated work, and it does not add new untracked files.

Write commit messages in English using [Conventional Commits](https://www.conventionalcommits.org/en), for example `feat(decoder): Add lookup table for distance 3` or `docs(board): Clarify serial console setup`. Keep the subject short and imperative. Add a body only when it explains something useful beyond the subject.

### Create a pull request

Keep pull requests as small and focused as possible. A PR that changes one module or adds one guide is much easier to review than a massive commit spanning half the semester.

After committing your work, bring your branch up to date with `main`:

```sh
git checkout main
git pull --ff-only
git checkout '<your-branch-name>'
git merge main
```

If there are conflicts, resolve them and review the result before continuing. Run the relevant checks, then push your branch:

```sh
git push -u origin '<your-branch-name>'
```

Open a pull request on GitHub. Keep the description brief: enough context for someone to understand the change, plus a related issue link if useful. For technical procedures, mention what you tested and any limitations. There is no rigid description template.

### Address review comments

Code reviews are collaborative check-ins, not tests.

* If a reviewer asks questions or suggests improvements, feel free to discuss them either directly on GitHub or in person.
* If you discuss things in person, write a brief comment on the PR summarizing what was agreed on so the decision is recorded.
* If you are self-reviewing your own pull request, use the PR diff view to check for accidental whitespace changes, leftover debugging statements, or missing files before merging.

### Merge your pull request

Once the change is ready, address outstanding review concerns and check any validation results available for the repository. Do not assume every repository has automated checks or required approvals.

* Generally, it is helpful for the author to merge, so they control when the change lands.
* Choose a merge method agreed for the repository; this guide does not mandate one. If using squash merge, review the resulting commit message and keep it consistent with the commit convention.

### Delete your branch

After merging a pull request, delete its branch when it is no longer needed. Before deleting it on GitHub or locally, confirm that all intended changes reached `main` and that there is no later work or work someone else still needs on the branch.

Cleaning up completed branches keeps it clear which work is active. Once you have confirmed this and deleted the remote branch on GitHub, clean up your local machine:

```sh
git checkout main
git pull --ff-only
git remote prune origin
git branch -d '<your-branch-name>'
```

Prune first so a stale upstream tracking reference cannot satisfy `git branch -d`'s merged check in place of the updated `main`.

After a squash merge, `git branch -d` may refuse because the original branch commits are not ancestors of `main`. This is a safety check. Confirm that the PR was merged, all intended changes reached `main`, and there is no later work on the branch before considering force deletion with `git branch -D`. If unsure, leave the branch in place and ask. Do not use force deletion as the default cleanup command.

## Shared practices

### Rebasing on shared branches is discouraged

Avoid rebasing branches that are shared or actively under review unless you coordinate with the people involved. Rewriting commits can disrupt their work, confuse review history, and replace signed commits with new ones.

The examples use `git merge main` to update a branch without rewriting its existing commits. Avoid force-pushing shared history. Check the repository's actual branch protections and rulesets; this guide does not claim that direct pushes or force-pushes are automatically blocked.
