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

#### Linting

**Avoid creating disable comments in the first place**, if practically possible.

```{note}
We recognize that sometimes, it's preferable to use disable comments. In particular, we don't think it's a good idea to restrict ourselves to rules that are completely universal.
```

If you do need a disable comment, **annotate it**, stating why it doesn't apply and/or is necessary, 
and **format** it **like this**: `# noqa: <code> [<rule-name>] - <annotation>`

````{admonition} For example
Instead of this:
```python
def spam():
  User = get_user_model()  # noqa: N806
```
Do this:
```python
def spam():
  User = get_user_model()  # noqa: N806 (=non-lowercase-variable-in-function) - this represents a class
```
````
```{note}
- We are deliberately **not using `ruff: ignore` syntax for portability**.
- We deliberately **include the code** with the human-readable name **for searchability** (search results for `S101` are more useful than for `assert`!)
```

### JavaScript

Please adhere to [Airbnb's JavaScript Style Guide](https://github.com/airbnb/javascript).

#### Linting

Use [ESLint](https://eslint.org) in your project setup to check code quality and detect errors as well as potential problems in the JavaScript code.

When using ESLint please also follow [the general linting guidelines](#linting-general), and add the additional rules to your `eslintrc.js` file:

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

Read and implement [the development section of the git guide](./using_git.md#development).

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
