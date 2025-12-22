# Development Guide

## Coding conventions

### Python

Follow the [PEP 8 Style Guide](https://www.python.org/dev/peps/pep-0008/) for Python Code.

- Naming style:
  1. Use lowercase for variable names.
  2. Use UPPERCASE for constants.
  3. Use lower_case_with_underscores for method names.
  4. Use lowercase for module names.
  5. Use CapWords convention for Class names.

Use pre-commit with ruff, uv, …

Configuration files can be found [here](https://github.com/base-angewandte/config) and included in a new project like this:

```bash
# include config in project
git subtree add --prefix config git@github.com:base-angewandte/config.git main --squash

# symlink the configuration files
ln -s config/.cz.toml .cz.toml
ln -s config/.hadolint.yaml .hadolint.yaml
ln -s config/.pre-commit-config.yaml .pre-commit-config.yaml
ln -s config/uv.toml uv.toml

# if you are using the python version configured in config/.ruff.toml
# you can just link it
ln -s config/.ruff.toml .ruff.toml

# otherwise you should extend from it
cat <<EOT >> .ruff.toml
extend = "config/.ruff.toml"

src = ["src"]
target-version = "<python version, e.g. py311>"
EOT

# create Makefile
# change <project_name> to the name of the project
cat <<EOT >> Makefile
include .env
export

PROJECT_NAME ?= <project_name>

include config/base.mk
EOT

# create gitignore
make gitignore
```

In case you need to adapt a certain config file, remove the symbolic link and copy the config file to use it as a template.

#### base style decisions

In some cases we stumble across style issues that are not clearly resolved or mandated by the PEP 8
style guide. In the following cases we came up with our own code style conventions:

- **string interpolation**: generally we use [f-strings](https://peps.python.org/pep-0498/) to do
  string interpolation, if there is no explicit reason to use another form
- **string interpolation in logger calls**: we also use f-strings for logger calls<br>

  ```{admonition} Background info regarding logging performance
  :class: hint dropdown

  whenever we log something, e.g. by using `logger.warning()` or any of the other logger methods,
  it would be good to not use f-strings for string interpolation but logger's
  `%s`-style syntax for performance reasons. This way the string interpolation happens at the latest
  possible time before the logger actually writes out the string.
  See [Python 3 Logging HOWTO: Optimization](https://docs.python.org/3/howto/logging.html#optimization)
  for more background info.

  This would be our preference. But [pyupgrade](https://pypi.org/project/pyupgrade/) does not work well
  with this (%-style is only kept if there’s just one argument). Therefore, we stick to using f-strings.
  If we ever run into performance issues in logging, we will work around this using %-style.
  ```

- **string interpolation for quoted values**: in cases where we want to have a value quoted, eg.
  as in `f'The value is "{value}".'`, we use the `repr()` function instead, so the f-string becomes
  `f'The value is {repr(value)}.'`. This results in single quotes around the value. Only in cases
  where double quotes are explicitly required, the initial form might be used, by applying the
  `# noqa: B907` comment to tell the linters to ignore this.

### JavaScript

Please adhere to [Airbnb's JavaScript Style Guide](https://github.com/airbnb/javascript).  
Use [ESLint](https://eslint.org) in your project set up to check code quality, detect errors and potential problems in the JavaScript code.  
When using ESLint please also add the additional rules to your `eslintrc.js` file:

```javascript
  rules: {
    // don't require .vue extension when importing
    'import/extensions': ['error', 'always', {
      js: 'never',
      vue: 'never', // specific for vue projects
    }],
    // disallow reassignment of function parameters
    // disallow parameter object manipulation except for specific exclusions
    'no-param-reassign': ['error', {
      props: true,
      // specific for vue projects
      ignorePropertyModificationsFor: [
        'state', // for vuex state
        'acc', // for reduce accumulators
        'e', // for e.returnvalue
      ],
    }],
    'vue/html-closing-bracket-newline': ['error', {
      singleline: 'never',
      multiline: 'never',
    }],
    // allow debugger and console during development
    'no-debugger': process.env.NODE_ENV === 'production' ? 'error' : 'off',
    'no-console': [process.env.NODE_ENV === 'production' ? 'error' : 'off', { allow: ['warn', 'error'] }],
  },
```

#### Vue.js

New base projects may be implemented using [Vue.js](https://vuejs.org/) or [NuxtJS](https://nuxtjs.org/) if server-side-rendering is required, respectively. Currently, all projects are using [Vue.js v2.x](https://vuejs.org/v2/guide/).
The project should be organized using [Single File Components](https://vuejs.org/v2/guide/single-file-components.html).

[Vue CLI](https://cli.vuejs.org/) may be used to set up new projects.

Vue ESLint configuration should include the [eslint-plugin-vue](https://eslint.vuejs.org/) with `plugin:vue/recommended` or [eslint-plugin-nuxt](https://github.com/nuxt/eslint-plugin-nuxt#readme) with `plugin:nuxt/recommended` as well as the respective Airbnb's base rule set ([eslint-config-airbnb.base](https://www.npmjs.com/package/eslint-config-airbnb-base) or [@vue/eslint-config-airbnb](https://www.npmjs.com/package/@vue/eslint-config-airbnb) via [Vue CLI](https://cli.vuejs.org/)).

#### Browserlist

Use following [Browserlist](https://github.com/browserslist/browserslist) configuration:

```
> 1%
last 2 versions
not dead
```

### HTML/CSS

Follow the [Google HTML/CSS Style Guide](https://google.github.io/styleguide/htmlcssguide.xml).

Use [Sass (dart-sass)](https://sass-lang.com/) as CSS preprocessor. This allows for the direct usage of the following base styleguide compatible scss variables:

```scss
/* fonts */
$font: "Source Sans Pro", serif;
$font-root-regular: 19px;
$font-root-mobile: 16px;
$font-size-regular: 1rem; // 14pt on desktop and tablet,
$font-size-small: 16rem/19;
$font-size-large: 24rem/19;
$line-height: 24rem/19;

/* height and spacing */
$spacing: 16rem/19;
$spacing-small: 8rem/19;
$spacing-large: 32rem/19;
/* new styleguide: font size + relative spacing! */
$row-height-large: 1rem + 2 * 1rem;
$row-height-small: 1.1rem + 2 * $spacing-small;
$header-height: 55px;
$chips-spacing: 0.05rem;
$chips-list-height: 1rem + 2 * $spacing;
$input-field-line-height: 27rem/19;
$fade-out-width: 30px;

/* colors */
$app-color: #000000; // replace this with the app color in question
$app-color-secondary: #ffffff; // replace this with the secondary app color in question
$font-color: rgb(0, 0, 0);
$font-color-second: rgb(111, 111, 111);
$font-color-third: rgb(160, 160, 160);
$button-header-color: rgb(240, 240, 240);
$input-field-color: rgb(200, 200, 200);
$background-color: #f0f0f0;
$box-color: white;
/* self defined for now! */
$uploadbar-color: #999999;

/* shadows and borders */
$border-width: 2px;
$border-active-width: 4px;
$box-shadow-reg: 0px 0px 3px 0px rgba(0, 0, 0, 0.05);
$box-shadow-hov: 0px 1px 5px 0px rgba(0, 0, 0, 0.15);
$drop-shadow: 0px 10px 10px 0px rgba(0, 0, 0, 0.25);
$pop-up-shadow: 0px 0px 36px 0px rgba(0, 0, 0, 0.5);
$pop-up-drop-shadow: 0px 10px 46px 0px rgba(0, 0, 0, 0.25);
$input-shadow: inset 0px 1px 4px 0px rgba(0, 0, 0, 0.2);
$preview-box-shadow: 0px 0px 16px 0px rgba(0, 0, 0, 0.25);
$box-transition: box-shadow 0.3s ease-in-out;
$separation-line: $border-width solid $button-header-color;
$list-border: 5px solid rgba(76, 175, 80, 1);
$list-border-transparent-50: 5px solid rgba(76, 175, 80, 0.5);
$list-border-transparent-100: 5px solid rgba(76, 175, 80, 0);
$upload-border: $border-width dashed $font-color-second;
$upload-border-hover: $border-width dashed $app-color;
$active-border: $border-active-width solid $app-color;
$input-field-border: 1px solid $input-field-color;
$loading-background: rgba(255, 255, 255, 0.5);

/* sizing */
$page-max-width: 1400px;
$page-min-width: 305px;

/* icons */
$icon-max: 48px;
$icon-large: 24px;
$icon-medium: 16px;
$icon-small: 12px;
$icon-min: 8px;

/* break-points */
$breakpoint-small: 640px;
$breakpoint-medium: 1024px;

/* z-indexes */
$zindex: (
  notification: 1100,
  // just in here for reference
  header: 1040,
  loader: 100,
);

/* transitions */
$page-transition-duration: 250ms;
```

### Accessibility

Whenever possible follow the [Web Content Accessibility Guidelines (WCAG) 2.1](https://www.w3.org/TR/WCAG21/) ([Quick Reference](https://www.w3.org/WAI/WCAG21/quickref/)) up to level AA.<br>
Use tools like [aXe](https://www.deque.com/products/axe/), [AChecker](http://achecker.ca/) or [Accessibility Insights for Web
](https://accessibilityinsights.io/docs/en/web/overview) to detect errors and potential problems. Additional tools can be found [here](https://www.w3.org/WAI/ER/tools/).

### Git

Follow the [git-flow branching model](http://nvie.com/posts/a-successful-git-branching-model/).<br>
A git extension can be found [here](https://github.com/petervanderdoes/gitflow-avh) and a short introduction of it [here](https://jeffkreeftmeijer.com/2010/why-arent-you-using-git-flow/).

The main branch should be `main` instead of `master`.

Additional to the established git-flow branches, `fix` branches can be used similar to `feature` branches,
for working on bugfixes that should be reviewed before being merged into `develop` again.

When merging a git-flow branch, **always create a merge commit, even if a fast-forward merge is possible, and even if your branch only has one commit**.

Check the [](./local_dev_setup.md) section for instructions on how to configure `git merge` and the `git-flow` extension to do the right thing by default here.

Alternatively, you can simply use the `--no-ff` flag on `git-flow` subcommands as well as vanilla `git merge` - for example, `git flow feature finish --no-ff` and `git merge --no-ff feature/add-shrubberies` would both always create a merge commit.

#### Commit Guidelines

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

#### Commit Message Guidelines

Use the [Conventional Commits specification](https://www.conventionalcommits.org/en/v1.0.0/) for your commits. The commit message should be structured as follows:

```
<type>(<optional scope>): <subject>

<optional body>

<optional footer(s)>
```

If the commit contains a **breaking change**, append a `!` after the `type`/`scope`.

The first line of the commit message containing `type`, `scope` and `subject` should be 50 characters or less. The maximum length of body lines should not exceed 72 characters.

##### Types

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

##### Scope

The scope provides additional contextual information.

- Is an **optional** part of the format
- Allowed scopes depend on the specific project, e.g. in the `base-ui-components` repository use the name of the component in question
- Don't use issue identifiers as scopes

##### Subject

The subject contains a succinct description of the change.

- Is a **mandatory** part of the format
- Use the imperative, present tense: "change" not "changed" nor "changes" as defined by the [git guidelines](https://git.kernel.org/pub/scm/git/git.git/tree/Documentation/SubmittingPatches#n181)
- Don't capitalize the first letter
- Do not end the subject line with a period

##### Body

A longer commit body can be provided after the subject, providing additional contextual information about the code changes.

- Is an **optional** part of the format
- Use the imperative, present tense: "change" not "changed" nor "changes"
- The body must begin with one blank line after the description

##### Footer

The footer should contain any information about **breaking changes** and is also the place to **reference issues** that this commit refers to.

- Is an **optional** part of the format
- Optionally reference an issue by its ID (`#<Issue ID>`)
- Breaking changes should start with the word `BREAKING CHANGE:` followed by a space or two newlines. The rest of the commit message is then used for this.

##### Examples

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

### Testing

#### Browser Support

##### Tier 1

All base services have to support Tier 1 browsers. They need to be tested during development and all bugs related to them have to be fixed.

**Desktop:**

- Chrome: latest
- Edge: latest
- Firefox: latest
- Safari: latest

**Mobile:**

- Chrome for Android: latest
- iOS Safari: latest

##### Tier 2

Tier 2 browsers are not actively supported. base services should work on them, but they are only tested infrequently and only severe bugs are fixed.

**Desktop:**

- Chrome: penultimate
- Edge: penultimate
- Firefox: penultimate
- Safari: penultimate

**Mobile:**

- iOS Safari: penultimate

##### Tier 3

Unsupported browsers.

- All other browsers not listed in Tier 1 and 2

### References

- https://wiki.openstack.org/wiki/GitCommitMessages
- https://semver.org/
