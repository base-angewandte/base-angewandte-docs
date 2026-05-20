# Using Git and Forgejo

There is a lot of information about how we use Git and Forgejo, so it all lives
in one place here in order to be easy to find, and avoid bloating the relevant
parts of the other sections.

```{tip}
The other sections refer to this document, so you may want to read them first
and return here with more context.
```

```{important}
Just be aware that you should definitely do so and **become familiar with our
workflow before contributing**, especially as it's tailored to our specific
requirements, and therefore likely not what you expect.
```

## Philosophy and informal tl;dr

As a quick intro and contextualization:

- We have development instances that reflect the latest state of development,
  which we also use for user acceptance testing. Production instances run code
  that has been deemed ready for production deployment.
  - This means that despite usually being overkill for developing rolling
    release web applications, **we implement `git-flow`** as there are concrete
    benefits for us.
  - It also means that while it's somewhat less critical that `develop` be in a
    perfect state than `main`, it **should be treated as a production branch**
    just the same.
- We see `git` as a tool to create a curated and easy to understand history of
  changes, not as a tool to document the development process.
  - This means that we implement a **workflow based on `git rebase`**, and don't
    care about rewriting history for changes that are still in development.
  - **`main` and `develop`** are of course treated as **immutable history**.
- Our test coverage is (depending on the project) far from complete at best,
  nonexistent at worst. We are working on CI infrastructure, but as of 2026-02,
  it is in its infancy.
  - This means that **we rely on `pre-commit` for QA**. It's not just a tool to
    avoid having to make changes due to CI failing, it **IS** currently our **only automated
    line of defense** against common and easily preventable errors. While that is still the case, **using it is essential**.
- Internally, for code review, we follow a **trust but verify** principle that assumes **good intentions but also human fallibility**.
  Our safeguards are circumventable, but only intentionally and explicitly, and in a way that leaves a paper trail for accountability.
  - For example, branch protection exists, but everyone has the ability to temporarily disable it.
  - And in general, we recognize that while rules have a purpose, following them to the letter when they aren't useful for that purpose (or another concern overshadows that purpose) can be detrimental. So, they aren't strictly enforced by technical means, but if they are broken, that needs to be justified, and there had better be a good reason for it.

```{note}
You can think of the last point as [**EAFP over LBYL**](https://realpython.com/python-lbyl-vs-eafp/) applied to code review.
```

## Setup

Before committing, make sure to check/update your git configuration:

- You should at least update your **user name** and **email** to what you would like to show up in the published commits.
- Our default branch is `main`, which might not be the case with older git versions.
  - But **you probably want to check out `develop`**, as that represents the newest state of development; `main` is for releases
- We follow the **`git-flow` branching model**. Check [](#development) for more details.
  - For `feature`/`fix`/`hotfix` branches, we use `rebase`; **do NOT merge the target branch (or any other branch) back into the branch you are working on!** Instead, use `git rebase target-branch`. Again, see [](#development) for more details.
  - **Always use the Forgejo/GitHub UI for merging these branches!**

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

Follow the [git-flow branching model](http://nvie.com/posts/a-successful-git-branching-model/), with a rebase-based workflow as documented below.

The main branch should be `main` instead of `master`.

Additional to the established git-flow branches, `fix` branches can be used similar to `feature` branches,
for working on bugfixes that should be reviewed before being merged into `develop` again.

For the rest of this document, unless explicitly specified otherwise, the term "feature branch" refers to all `feature`/`hotfix`/`fix` branches.

#### Use `rebase`

When working on a feature branch:

- **do not merge the target branch back into your branch**.
- Instead, **rebase on the target branch**.
  The **only exception** to this rule is if you are working on a feature branch where such a merge happened **before the rule got introduced**, and you would have to replay a merge commit while rebasing.

```{important}
**Collaborating on the same branch is NOT an exception to this rule**, see [](#collaboration).
```

```{tip}
##### An awesome git tutorial

If you're not comfortable with using rebase yet, consider following [this awesome interactive tutorial](https://learngitbranching.js.org/) to become familiar with the concept! If you're already an experienced git user, the **relevant lessons for `rebase` (and `cherry-pick`)** are:

- Main: Introduction Sequence:
  - Level 4: Rebase Introduction
- Main: Moving Work Around:
  - All levels
- Main: A Mixed Bag:
  - Level 1: Grabbing Just 1 Commit
  - Level 2: Juggling Commits
  - Level 3: Juggling Commits #2
- Main: Advanced Topics:
  - Level 1: Rebasing over 9000 times
  - Level 3: Branch Spaghetti
- Remote: Push & Pull -- Git Remotes!:
  - Level 7: Diverged History
- Remote: To Origin And Beyond -- Advanced Git Remotes!
  - Level 1: Push Main!
  - Level 2: Merging with remotes _(only for contrast with the rebase approach, and perhaps as a demonstration of why we use the rebase approach)_
```

#### Rebase continuously

Regularly rebase feature branches on the target branch during development.

At bare minimum, you must **rebase before code review**, and again **before performing the merge**.

This is because unless branches are based on the tip of your target branch, once merged, the tip won't be the code you tested, but a merge product - and even with a clean rebase with no conflicts, there could be regressions.

```{note}
External contributors who aren't comfortable with rebase need not do this.
```

#### Collaboration

When collaborating on a branch (or simply pulling in changes made as part of the review process), use the same workflow:

- Instead of `git pull`, use `git pull --rebase`.
  - You may want to alias this to `git pure`.
- Instead of `git push` use `git push --force-with-lease`.
  - You may want to alias this to `git please`.

```{note}
This is a relaxation of the often-cited "golden rule of rebasing" not to ever rebase a public branch.

**This is intentional**, and because we recognize that as long as `pull --rebase` and `push --force-with-lease` are consistently used, the workflow is equivalent - and doesn't lead to merge commits in feature branches, which we do not want to have.
```

#### Merging

When merging a feature branch, **always use the Forgejo (or GitHub) UI** to merge a PR.

The only exception is when working on an old branch that the target branch was merged back into before the policy change introducing the rebase workflow.

```{warning}
If it happened later, I hope you like re-resolving any conflicts and also maybe [picking cherries](https://git-scm.com/docs/git-cherry-pick).
```

In that case, you will have to temporarily disable branch protection on the target branch to allow manual pushes.

#### Create feature branches based on `main` or `develop`

ie: **Do not create feature branches based on other feature branches**.

This rule is less strict than the other ones. Developing a set of related experimental changes in tandem with each other locally can be a great way to realize how these changes will interact in practice, and later factoring these changes into multiple PRs with a smaller scope is actually a very good idea.

However, **never open a PR with a feature branch as a target**.

Instead, if you want to make a PR targeting a feature branch:

- open the PR as a WIP targeting `main` or `develop` to keep track of it
- only open it for review once it only contains the changes that are in-scope for the PR

```{tip}
Just because the _PR_ needs to target `main` or `develop` doesn't mean that you can't still base _your branch_ on another branch!

If you do this, make sure to regularly rebase it on the feature branch in the meantime.
```

##### Prioritize code review over chaining branches

If this feels restrictive and like it blocks you from getting work done, **put your energy into code review instead**, contributing to an environment where code review happens quickly and is less of a bottleneck.

If you request code review from a colleague and get a request for review from them more than once before they review your code, feel free to call them out on that. (Also, don't be that colleague.)

#### How to implement this workflow

If you are used to working with conventional merges, then this workflow may seem intimidating. Fortunately, there are many parallels.

##### Cheat sheet

- Instead of `git merge <target-branch>` to get recent changes from the target branch, do `git rebase <target-branch>`
```{note}
Be aware that because of how `rebase` works, this might give you more than one set of conflicts, one for each commit in your branch, in contrast to `merge`, where you resolve everything at once.
```
- Instead of `git pull` to get new commits others made in a feature branch, do `git pull --rebase`
- Instead of `git push` to push your changes to a feature branch, do `git push --force-with-lease`
```{danger}
Do **NOT** do `git push --force` **EVER**! If `git push --force-with-lease` didn't work, that would overwrite someone else's work in 99% of cases.
```

```{tip}
There are some aliases in [](#setup) that we'd recommend for ergonomics! And of course, you can also define your own.
```

You should understand what a rebase does instead of following this cheat sheet blindly, but rest assured, it's not black magic, and not much actually changes.

If you don't understand rebase yet, see the [](#use-rebase) subsection above for an interactive tutorial.

#### Hotfixes

For [hotfixes as defined by `git-flow`](https://nvie.com/posts/a-successful-git-branching-model/#hotfix-branches), there are two possibilities:
- **time-critical hotfixes** (security issues, significant features not working):
  - Make a hotfix branch based on `main` and create a PR **that you merge immediately without review**
  - Do NOT merge `main` back into `develop`
  - Instead, make a branch that contains the same commit(s) as the hotfix, but rebased on `develop`, and make a PR for that
  - Go through the standard review process for this PR, backporting any resulting changes to `main` in a new hotfix branch
    - This branch obviously does not require review or a followup, unless it doesn't apply cleanly
- **other hotfixes**:
  - Make a hotfix branch based on `main` and create a PR **that gets reviewed normally**
  - Once this review is done, make a branch that contains the same commit(s) as the hotfix, but rebased on `develop`, and make a PR for that
  - If the commits apply cleanly to `develop` (rebase is successful, maybe with some trivial merge conflicts, no other changes): merge this PR yourself
  - Otherwise, go through the standard review process for this PR
- In both cases:
  - Link to the hotfix PR in the PR to `develop`

```{note}
We also use "hotfix" branches for things that aren't critical, but should be deployed ASAP/sooner than the current state of `develop` allows for, like typos, missing tranlations, wrong info, fixes for usability issues, etc
```

```{note}
The multiple PRs may seem redundant, but they document what happened and also provide a sanity check via CI (once we have that)
```

```{tip}
For rebasing hotfix branches on develop, [`git cherry-pick`](https://git-scm.com/docs/git-cherry-pick) is a simple solution.

Hotfix branches usually shouldn't be too large, but here's an alternative if there are many commits:
```sh
git checkout hotfix/<branch-name>
git checkout -b hotfix/<branch-name>-develop
git rebase --onto develop main
```

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

## Review process

As the review process ties in closely with using Git, it is documented here.

Internally, we use Forgejo, and a trust-based workflow. External contributors should read the [](#external-contributors) subsection.

```{note}
This section is meant as a rough guideline, not absolute gospel. Take it with a grain of salt, and adjust for context - just be prepared to explain why you did so.
```

### Goals of code review

There are multiple goals of code review:

- **Quality assurance:** providing a second, unbiased look at the proposed changes to
  - spot more issues before they can pop up in production
  - uphold code quality standards
- **Horizontal transfer of knowledge/skills** within the team
- **Documenting the development process** - PRs are an excellent source of information for future developers wanting to understand **not just which changes** were made, but **what considerations** led to that specific solution
- **BUT ALSO:** Getting the code merged!
  - And ultimately, **this is the primary goal!**
  - The **other goals** serve to implement **prerequisites** for that to happen, or provide some **additional benefit at little extra cost**
    - For example, if quality standards aren't met, the code shouldn't be merged, making QA a prerequisite goal
  - **Where pursuing another goal doesn't do that, if only for a future review, it is detrimental to the review process**

### External contributors

External contributors can **submit patches via GitHub**, for the projects that are mirrored there.

If you are an external contributor: Welcome! Please **read the rest of this section**, even though not everything will apply to you, and treat this subsection as a list of differences.

Obviously, we can't give you the same level of access as team members, much of the next subsection doesn't apply, and we'll use the **traditional open source model**.

Not everyone is equally comfortable using `rebase`. If you aren't:

- Consider doing [this interactive git tutorial](https://learngitbranching.js.org/) to familiarize yourself with how it works
  - The [](#use-rebase) section above has a list of levels relevant to `rebase`!
- Whatever you do, **DO NOT merge the target branch back into your own branch**
  - PRs with such merge commits won't be accepted
- If you need to replicate the effect of doing so, there's no way around performing a simple rebase. However:
  - it should be as simple as (while on your branch)
    - `git fetch origin develop:develop` (or `git fetch origin main:main`)
    - `git rebase develop` (or `git rebase main`)
    - `git push --force-with-lease`
  - if it isn't that simple, ask us for help!
- If you're done and the only thing left to do is to rebase before merging, let us know and we'll do that for you!

### Trust-based model

```{note}
This doesn't apply to external contributors.
```

As we are a team of professionals, we explicitly depart from the traditional open source model in that there is **no strict separation between contributor and maintainer**, or author and reviewer in general. Instead, we treat **code review as a trust-based collaborative process**. While the author of a PR is responsible for getting the changes merged by default, and often will be the sole contributor of concrete code changes in the end, this is for practical reasons rather than because of policy.

#### Some general principles

- We **trust each other as professionals** to make correct calls taking into account the available info that was considered.
- We **do NOT trust** that, as we are fallible humans, all relevant **info WAS available, or considered**.
- So: We implement safeguards, but these safeguards are soft and intentionally circumventable - **but only by intentional and explicit action**.
  - For example, branch protection exists, but everyone has permission to temporarily disable it.
  - Basic assumption: **if it can be forgotten without noticing that anything is wrong, it will be**.
- In general, when it comes to code review, we recognize that while rules have a purpose, **they are only useful if they actually serve that purpose, and it can be in the interest of common sense to break them**, based on professional opinion and taking context into account. Furthermore, doing so **unilaterally** may result in disagreements, but resolving those if they arise can be significantly **less overhead** than invoking a feedback loop every time.
  - For accountability regarding this point, and to prevent [normalization of deviance](https://en.wikipedia.org/wiki/Normalization_of_deviance), **deviations from standard procedure must be documented in writing where they will get noticed**.
  - Also, anyone doing so must be prepared to justify their decision to other team members if asked.
  - This documentation must be in plain view of any team member viewing records of the actions taken (for example, in the same PR).
  - As an example, if a team member makes a review requesting changes, and that review gets dismissed, the "Reason" field should be used to explain in detail why the review was dismissed.

```{note}
You can think of the last point as [**EAFP over LBYL**](https://realpython.com/python-lbyl-vs-eafp/) applied to code review.
```

### Workarounds

```{note}
This only partially applies to external contributors.
```

Unfortunately, Forgejo is somewhat geared to that traditional open source model, and a bit all-or-nothing regarding safeguards. That means that we need some workarounds.

#### We're all admins

In order to make sure that we have the final say and can do what we need to, we're all repository admins for our projects.

#### Admins can bypass branch protection

This means that even if there are blockers that should prevent a merge, the UI will offer you to do a merge anyway.

```{important}
**DO NOT DO THIS without a very good reason**.
```

If the merge button is red, and you think that merging is necessary for some reason:

- first, reconsider.
- then, if you're sure regardless:
  - unless it's extremely time-critical (like a critical security hotfix): put a brief message in the team chat, giving everyone who is present at least 15 minutes to raise concerns.
  - for every blocker Forgejo shows:
    - document in a comment why you are ignoring that blocker and merging anyway.
    - remove the blocker if possible (eg. by disregarding reviews) in case Forgejo isn't showing all of them at once.
  - do a manual check for any unresolved comment threads, and resolve them with a comment stating why.
  - double-check the team chat for replies to your message
  - unless there are objections, perform the rebase

#### Treat unresolved comment threads as blockers

Forgejo won't treat unresolved comment threads as blockers, only code reviews requesting changes.

You should treat them as blockers anyway, even if they are not attached to a review requesting changes by mistake.

#### Always request changes if you are leaving a resolvable thread

This is so that the author sees a blocker and doesn't accidentally merge.

On the other hand, feel free to dismiss reviews requesting changes if and only if all threads it introduced are resolved, and the review comment itself has been addressed.

#### Use resolvable review comments

Use resolvable review comments extensively.

Comments only result in a resolvable item if they are made on the diff of the PR somewhere.

**So, do that, even if it doesn't relate to a specific change in the diff, you can use a random location** - and note in your comment that it is unrelated to the diff.

```{admonition} Some examples for comments that should be blockers that don't relate to a (specific) current line in the diff

- "We need PR #42069 to be merged before merging this. Before resolving, make sure that this PR is merged and this branch is rebased on current `develop`"
- "I have doubts about `<X general design decision spanning many specific changes>`, what is your reasoning, and what alternatives did you consider?"
- Any comments about the absence of important changes, like:
  - "This PR needs tests"
  - "You forgot to make a migration for your model change"
- "I am worried about a regression with `<Y feature that is implemented somewhere completely different>`"
- "This PR is just one huge commit, please do an interactive rebase, split up the commit into multiple smaller ones, and make it reviewable"
```

#### Resolving comments

The default assumption when a reviewer creates a resolvable thread is that the reviewer will resolve it when the issue it raises has been addressed to their satisfaction.

Deviations from this are made explicit.

```{admonition} Some examples
- "I think this would be more readable as a list comprehension, but that's a matter of taste, feel free to resolve if you disagree"
- "I think X change would be a good idea here, but also, that's out of scope for this PR - feel free to resolve after making a followup issue"
- "This PR needs <other PR> to be merged first, if that's happened, feel free to resolve this"
```

##### Notify reviewers when you're done

As an author, notify reviewers when you've implemented the changes that were requested by replying to the thread.

#### Comments by authors

Comment threads can also be made by the author of a PR if they notice something on their own. These threads can be resolved by the PR author, and the list of unresolved comments used as a todo list.

### Merging

When merging, **always do a rebase + merge commit via the Forgejo UI**.

Do not merge PRs without review from at least one colleague, unless it's an urgent hotfix.

Merges are done by the reviewer by default, but the reviewer can give conditional consent otherwise.

```{admonition} Examples for conditional consent
- "Apart from the missing migration and the typo in the docs, LGTM. Feel free to merge on pipeline success once that is done."
- "LGTM, but it's Friday afternoon, let's not merge this now. I'm on vacation next week, but feel free to merge first thing Monday morning!"
```

### Using issues

You're encouraged to use issues, especially if you aren't starting to work on a task yet. However, you're also free to make WIP PRs for branches right from the start (or even as placeholders for future development) so that all relevant info can be in one place, especially if you'd like to use the PR to eg. make notes on the diff/run CI/have an overview of your changes.

```{note}
Bear in mind that at the time of this writing, it's impossible to filter PRs by their WIP status in Forgejo, so avoid cluttering the project with too many of them.
```

### WIP PRs

PRs that **aren't ready for review yet should be marked as WIP**. It's encouraged to mark PRs as WIP again if something comes up that means that they shouldn't be merged in their current state, however, this isn't required - you can also simply create a blocker/review comment. Just ensure somehow that your colleagues won't accidentally merge the PR.

Before being marked as ready for review for the first time, there are **no rules for the content of WIP PRs**. They can contain experimental changes, 100 unsquashed microcommits that lead to a +2/-1 diff, no changes at all, 100 slightly different instances of the same +1728/-1337 change that has been force-pushed with slight alterations until CI passed, all of the above over the course of their lifespan, or whatever else supports the personal workflow of the authoring team member.

Unless a colleague has requested a pre-review of an almost-ready WIP PR, do not complain about what you see when you make the choice to look at it.

```{note}
**We are accountable for what we choose to present, not how we get there**.
```

After being marked as ready for review for the first time, PRs should at least more or less follow standards of reviewability, even when put back into WIP state while changes are being made.

Respect the fact that your fellow team members might now have a starting point/mental model to look at future changes to the PR from, and don't completely alter the history (although squashing changes into the commits they should have been a part of initially is still a really good idea).

### Standards of reviewability

PRs marked as ready should

- target `main` or `develop`
- not contain commits that are to be merged as part of another PR

All commits should pass `pre-commit` checks, unless they are part of a WIP while changes are being made after review.

CI should already have passed, or changes since the last time CI passed should be minimal, and the CI jobs queued or running.

```{note}
For projects that don't have CI yet, consider that any given statement is true for all members of the empty set, and draw your conclusions.)
```

The PR branch should be rebased on its target at the time the PR is marked as ready.

History should be in a reviewable state. This is subject to author ability, but ideally:

- If large changes were made and then undone/redone, these changes should probably be squashed
  - This is so that reviewers that go commit by commit don't have to read multiple versions of the same changes, and delete pending comments as they realize that their requested change was already made in a later commit
  - The same applies to future developers trying to understand the history
  - Remember: Our goal is to create a curated and easy to understand history, not to document the development process
- Changes should follow the [](#commit-guidelines)

The scope of the changes should be minimal. If the PR is decomposable into two PRs that make sense individually, then it should be split up.

The PR should not make test coverage worse:

- if new behavior is implemented, it should be covered by tests
- if existing behavior is changed significantly, and it wasn't covered by tests before, tests should be added
- for complex refactors, tests should be added before the refactor happens, and pass on both the old and new version of the code
- trivial changes/refactors don't need new tests
- hotfixes never need tests, but depending on the nature of the hotfixes, they should be added in a followup PR

```{admonition} A caveat

Don't take this subsection too seriously, and use common sense. When in doubt, apply the spirit of the rules, not the letter, and account for context.

Ultimately, the goal here is to make code review less frustrating for everyone involved, not to add barriers.

Remember the goals of code review discussed above, and make sure that your actions are in line with them.
```

### Code ownership and etiquette

Including and especially when it comes to review, **there is no "your" code; there is only OUR code, and your contributions to it**.

You should be willing to embrace constructive criticism as a way to improve as a developer. In particular, technical concerns are ALWAYS more important than developer egos.

On the other hand, criticism should be constructive, and either impersonal or framed positively. Within those bounds, clarity of communication trumps politeness, and you should not minimize the potential negative outcomes of a proposed solution to make a colleague feel better. However, you should also not assume ill intent, and in general, **while you can and should be as _blunt_ as you need to be to convey your concerns, _don't be derogatory on a personal level_**.

When reading a review that feels harsh, re-read this section and remind yourself that your reviewer was probably following it.

When reviewing, make sure that you keep the goals of code review as discussed above in mind (especially the last one):

- Don't unnecessarily nitpick, and as far as possible, separate personal taste from technical concerns.
  - There's a fine line here, when walking it, also consider making comments but marking the suggested changes as optional/"feel free to disagree and just resolve".
  - If you think that someone might not have been aware of an applicable and generally useful approach, that should push you towards suggesting the change.
- In particular, don't block merges JUST for tiny improvements like eg. variable name changes or slight optimizations where performance isn't critical. If you have the choice to either merge a PR or request such a change, merge the PR, then make the change yourself in a new PR, allowing everyone else to base their work on the meat of the first PR.
  - If you do so, consider assigning the initial PR author as a reviewer
    - If that happens to you, be happy that your code got merged and you didn't have to make the change yourself!
  - If other changes are needed anyway, no harm in having some minor items as well
  - In general, weigh the value of making a change now against the value of merging the PR now

```{warning}
All of that said, **err on the side of speaking up if you think that a change shouldn't be merged in its current state**.
```

#### Disagreements

Remember that just because you're in the role of a reviewer, that doesn't make your opinion more valid than the author's.

```{important}
**You are collaborating, not gatekeeping.**
```

You can disagree with the author's chosen solution, but the author can disagree with your proposed alternative just the same.

If that happens, instead of trying to "win"™ the discussion, try to isolate what you fundamentally disagree about, and get a third opinion from another team member.

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
