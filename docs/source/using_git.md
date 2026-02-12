# Using Git and Forgejo

There is a lot of information about how we use Git and Forgejo, so it all lives
in one place here in order to be easy to find, and not bloat the relevant parts
of the other sections.

The other sections refer to this document, so you may want to read them first
and return here with more context. Just be aware that you should definitely do
so and become familiar with our workflow before contributing.

## Philosophy and informal tl;dr

As a quick intro and contextualization:
- We see `git` as a tool to create a curated history of changes, not as a tool
  to document the development process.
  - This means that we implement a **workflow based on `git rebase`**, and don't
    care about rewriting history.
  - The exception to this rule are **`main` and `develop`** (see below). These
    branches are treated as **immutable history**.
- We have development instances that reflect the latest state of development,
  which we also use for user acceptance testing. Production instances run code
  that has been deemed ready for production deployment.
  - This means that despite usually being overkill for developing web
    applications, **`git-flow` is useful to us, and we implement it**.
  - It also means that while it's somewhat less critical that **`develop`** be in
    a perfect state than `main`, it **should be treated as a production branch**
    just the same.
- Our test coverage is (depending on the project) far from complete at best,
  nonexistent at worst. We are working on CI infrastructure, but as of 2026-02,
  it is in its infancy.
  - This means that **we rely on `pre-commit` for QA**. It's not just a tool to
    avoid having to make changes due to CI failing, it IS currently our main
    line of defense. **Using it is essential**.
- Internally, for code review, we follow the principle that
  - We **trust each other as professionals** to make correct calls taking into
    account with the available info that was considered.
  - We **do NOT trust** that, as we are humans, all relevant **info WAS available, or
    considered**.
  - So: We implement safeguards, but these safeguards are soft and
    intentionally circumventable - **but only by intentional and explicit action**.
    - For example, branch protection exists, but everyone has permission to
      temporarily disable it.
  - And in general, we apply **EAFP over LBYL**. Rules aren't technically enforced,
    but if they are broken, there had better be a good reason for it.

## Setup

Before committing, make sure to check/update your git configuration:
- You should at least update your **user name** and **email** to what you would like to show up in the published commits.
- Our **default branch is `main`**, which might not be the case with older git versions.
  - But you probably want to check out `develop`, as that represents the newest state of development; `main` is for releases
- We follow the **`git-flow` branching model**. Check [](#development) for more details and a recommended git extension.
  - For `feature`/`fix`/`hotfix` branches, we use `rebase`; do NOT merge the target branch (or any other branch) back into the branch you are working on! Instead, use `git rebase target-branch`. Again, see [](#development) for more details.
  - Always use the Forgejo/GitHub UI for merging these branches!

Here is how to check and update the corresponding settings:

 ```bash
 # check your git config
 git config --global user.name
 git config --global user.email
 git config --global init.defaultBranch
 # update the settings (careful: applies to all repos!)
 git config --global user.name "Ms. Robot"
 git config --global user.email "ro@example.org"
 git config --global init.defaultBranch "main"
 ```

 ```{warning}
 Be aware that this will change your git config for all repositories - if you're a one-time contributor, you may want to run these commands only in the repository you want to contribute to, and leave out the `--global` flag.
 ```

 Alternatively you can also directly edit the _.gitconfig_ file in your home directory (e.g. with
 `editor ~/.gitconfig`). Here is a template including some handy shortcuts for git:

 ```ini
 [user]
     email = ro@example.org
     name = Ms. Robot
 [alias]
     co = checkout
     ci = commit
     st = status
     br = branch
     hist = log --pretty=format:\"%h %ad | %s%d [%an]\" --graph --date=short
     type = cat-file -t
     dump = cat-file -p
     please = push --force-with-lease
     pure = pull --rebase

 [core]
     # leave this one out, if you want to keep the standard (nano), or change
     # to your preferred editor
     editor = vim
 [init]
     defaultBranch = main
 ```

## Development

### Workflow

#### `git-flow`

Follow the [git-flow branching model](http://nvie.com/posts/a-successful-git-branching-model/), with a rebase-based workflow as documented below.<br>
A git extension can be found [here](https://github.com/petervanderdoes/gitflow-avh) and a short introduction of it [here](https://jeffkreeftmeijer.com/2010/why-arent-you-using-git-flow/).

The main branch should be `main` instead of `master`.

Additional to the established git-flow branches, `fix` branches can be used similar to `feature` branches,
for working on bugfixes that should be reviewed before being merged into `develop` again.

For the rest of this document, unless explicitly specified otherwise, the term "feature branch" refers to all `feature`/`hotfix`/`fix` branches.

#### Use `rebase`

When working on a feature branch:
- **do not merge the target branch back into your branch**.
- Instead, **rebase on the target branch**.
The **only exception** to this rule is if you are working on a feature branch where such a merge happened before the rule got introduced, and you would have to replay a merge commit while rebasing.

**Collaborating on the same branch is NOT an exception to this rule**, see [](#collaboration).

#### Rebase continuously

Regularly rebase feature branches on the target branch during development.

You must **rebase before code review**, and again **before performing the merge**.

This is because unless branches are based on the tip of your target branch, once merged, the tip won't be the code you tested, but a merge product - and even with a clean rebase with no conflicts, there could be regressions.

#### Collaboration

When collaborating on a branch (or simply pulling in changes made as part of the review process), use the same workflow:
- Instead of `git pull`, use `git pull --rebase`.
- Instead of `git push` use `git push --force-with-lease`.
  - You may want to alias this to `git please`.

Note that this is a relaxation of the often-cited "golden rule of rebasing" not to ever rebase a public branch. **This is intentional**, and because we recognize that as long as `pull --rebase` and `push --force-with-lease` are consistently used, the workflow is equivalent - and doesn't lead to merge commits in feature branches, which we do not want to have.

#### Merging

When merging a feature branch, **always use the Forgejo (or GitHub) UI** to merge a PR. Do NOT use the `git-flow` tooling to finish branches, as it will not only merge locally, but also delete your local branch.

The only exception is when working on an old branch that the target branch was merged back into before the policy change introducing the rebase workflow (if it happened later, I hope you like re-resolving any conflicts and also maybe [picking cherries](https://git-scm.com/docs/git-cherry-pick)). In that case, you will have to temporarily disable branch protection on the target branch to allow manual pushes.

#### How to implement this workflow

If you are used to working with conventional merges, then this workflow may seem intimidating. Fortunately, there are many parallels.

##### Cheat sheet

- Instead of `git merge <target-branch>` to get recent changes from the target branch, do `git rebase <target-branch>`
  - Be aware that because of how `rebase` works, this might give you more than one set of conflicts, one for each commit in your branch, in contrast to `merge`, where you resolve everything at once.
- Instead of `git pull` to get new commits others made in a feature branch, do `git pull --rebase`
- Instead of `git push` to push your changes to a feature branch, do `git push --force-with-lease`
  - Do **NOT** do `git push --force` **EVER**! If `git push --force-with-lease` didn't work, that would overwrite someone else's work in 99% of cases.

You should understand what a rebase does instead of following this cheat sheet blindly, but rest assured, it's not black magic, and not much actually changes.

##### Aliases over config

The previous version of these docs recommended configuring some git commands to do the right thing by default; however, with the new workflow, this makes less sense, the common behaviors aren't necessarily sane defaults (a `git pull` while on `develop` to pull in new changes should error if you accidentally added a commit there instead of a branch). In general, rebasing, while safe if done correctly, should be a conscious choice.

However, there are some aliases in [](#setup) that we'd recommend for ergonomics instead! And of course, you can also define your own.

### Commit Guidelines

The cardinal rule for creating good commits is to ensure there is only one "logical change" per commit. There are many reasons why this is an important rule:

- The smaller the amount of code being changed, the quicker and easier it is to review and identify potential flaws.
- If a change is found to be flawed later, it may be necessary to revert the broken commit. This is much easier to do if there are not other unrelated code changes entangled with the original commit.
- When troubleshooting problems using Git's bisect capability, small well defined changes will aid in isolating exactly where the code problem was introduced.
- When browsing history using Git annotate/blame, small well defined changes also aid in isolating exactly where and why a piece of code came from.

Avoid:

- Mixing whitespace changes with functional code changes.
- Mixing two unrelated functional changes.
- Sending large new features in a single giant commit.

The basic rule to follow is:<br>
**If a code change can be split into a sequence of patches/commits, then it should be split.**

### Commit Message Guidelines

Use the [Conventional Commits specification](https://www.conventionalcommits.org/en/v1.0.0/) for your commits. The commit message should be structured as follows:

```
<type>(<optional scope>): <subject>

<optional body>

<optional footer(s)>
```

If the commit contains a **breaking change**, append a `!` after the `type`/`scope`.

The first line of the commit message containing `type`, `scope` and `subject` should be 50 characters or less. The maximum length of body lines should not exceed 72 characters.

#### Types

The following types can be used:

- `feat` for commits that add a new feature
- `fix` for commits that fix a bug
- `security` for commits that fix a security issue
- `change` for commits that change the implementation of an existing feature
- `refactor` for commits that rewrite/restructure your code, however do not change any behaviour
- `prepare` for preparations of a feature
- `perf` for commits that improve performance
- `deprecate` for commits that deprecate existing functionality, but do not yet remove it
- `remove` for commits that remove a feature
- `revert` for reverting previous commits
- `style` for commits that do not affect the meaning (white-space, formatting, missing semi-colons, etc.)
- `test` for commits that add missing tests or correcting existing tests
- `docs` for commits that affect documentation only
- `build` for commits that affect the build system or external dependencies
- `chore` for miscellaneous commits e.g. modifying `.gitignore`

#### Scope

The scope provides additional contextual information.

- Is an **optional** part of the format
- Allowed scopes depend on the specific project, e.g. in the `base-ui-components` repository use the name of the component in question
- Don't use issue identifiers as scopes

#### Subject

The subject contains a succinct description of the change.

- Is a **mandatory** part of the format
- Use the imperative, present tense: "change" not "changed" nor "changes" as defined by the [git guidelines](https://git.kernel.org/pub/scm/git/git.git/tree/Documentation/SubmittingPatches#n181)
- Don't capitalize the first letter
- Do not end the subject line with a period

#### Body

A longer commit body can be provided after the subject, providing additional contextual information about the code changes.

- Is an **optional** part of the format
- Use the imperative, present tense: "change" not "changed" nor "changes"
- The body must begin with one blank line after the description

#### Footer

The footer should contain any information about **breaking changes** and is also the place to **reference issues** that this commit refers to.

- Is an **optional** part of the format
- Optionally reference an issue by its ID (`#<Issue ID>`)
- Breaking changes should start with the word `BREAKING CHANGE:` followed by a space or two newlines. The rest of the commit message is then used for this.

#### Examples

```none
feat(shopping cart): add the amazing button
```

```none
feat!: remove ticket list endpoint

Refs: #1337
BREAKING CHANGE: ticket enpoints no longer supports list all entites
```

```none
fix: prevent racing of requests

Introduce a request id and a reference to latest request. Dismiss
incoming responses other than from latest request.

Remove timeouts which were used to mitigate the racing issue but are
obsolete now.
```

```none
style: remove empty line
```

```none
build(deps): update pip-tools to 6.9.0
```

### Versioning

Given a version number MAJOR.MINOR.PATCH, increment the:

- MAJOR version for large feature changes,
- MINOR version for medium feature changes, and
- PATCH version for small improvements and bug fixes.

#### Further Guidelines

- Once a version has been released, the contents of that version MUST NOT be modified. Any modifications MUST be released as a new version.
- Major version zero (0.y.z) is for initial development. Anything may change at any time.
- Version 1.0.0 defines the first public release.

### Changelog

The changelog should follow the [Keep a Changelog guidelines](https://keepachangelog.com/en/1.0.0/) and you can map the conventional commit `type` to the change log headlines like this with the sections in the following order:

- `feat` → **Added**
- `change` → **Changed**
- `deprecate` → **Deprecated**
- `remove` → **Removed**
- `fix` → **Fixed**
- `security` → **Security**

## Tools

### BFG Repo-Cleaner

BFG can be used to remove sensitive data from a repository.

#### Usage

1. Install BFG, e.g. via homebrew for MacOS, or download the jar-File from https://rtyley.github.io/bfg-repo-cleaner/

2. BFG doesn't modify the latest commit (on `main` or `HEAD`), so ensure it is already clean.

3. Clone a fresh copy of your repo, using the `--mirror` flag:

   ```
   git clone --mirror ssh://git@github.com:base-angewandte/example.git
   ```

4. Clean Files and Strings.

   Examples for deleting files:

   ```
   bfg --delete-files id_{dsa,rsa}  example.git
   bfg --delete-files "file_name_*.py"  example.git
   ```

   Example for removing blob:

   ```
   bfg --strip-blobs-bigger-than 50M  example.git
   ```

   Example for replacing strings listed in a file (lines can be prefixed with `regex:` or `glob:` if required) with `***REMOVED***`:

   ```
   bfg --replace-text passwords.txt  example.git
   ```

   ```{note}
   If you are using the jar file, `bfg` is an alias for `java -jar bfg.jar` in all examples.
   ```

5. BFG will update your commits and all branches and tags so they are clean, but it doesn't physically delete the unwanted stuff. So you need to run the following to achieve that as well:

   ```
   cd example.git
   git reflog expire --expire=now --all && git gc --prune=now --aggressive
   ```

6. Push the cleaned repository:

   ```
   git push
   ```

   ```{note}
   This will update **all** refs on the remote server.
   ```

## References

- https://wiki.openstack.org/wiki/GitCommitMessages
- https://semver.org/
