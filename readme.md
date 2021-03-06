# [Argus Eyes](http://arguseyes.io/) [![npm version](https://badge.fury.io/js/argus-eyes.svg)](https://www.npmjs.org/package/argus-eyes)

| Linux & Mac OS X | Windows |
| --- | --- |
| [![build status](https://travis-ci.org/arguseyes/argus-eyes.svg?branch=dev)](https://travis-ci.org/arguseyes/argus-eyes) | [![build status](https://ci.appveyor.com/api/projects/status/github/arguseyes/argus-eyes?branch=dev&svg=true)](https://ci.appveyor.com/project/BranvanderMeer/argus-eyes) |

A lightweight commandline tool for visual regression testing of UI components.


## Contents

- [Overview](#overview)
- [Introduction](#introduction)
- [Guide](#guide)
  - [Install](#install)
  - [Step 1: Setup config file](#step-1-setup-config-file)
  - [Step 2: Take screenshots](#step-2-take-screenshots)
  - [Step 3: Test for visual differences](#step-3-test-for-visual-differences)
  - [Gitflow: Testing separate branches](#gitflow-testing-separate-branches)
- [Reference](#reference)
  - [Config](#config)
  - [Usage](#usage)
  - [Options](#options)
- [Contributing](#contributing)
- [License](#license)


## Overview
<!-- start -->
Argus eyes does 3 things for you:

1. Take screenshots of UI components in different views and branches
2. Test the screenshots for visual differences
3. Create images of the visual differences

Screenshots are caputured with [PhantomJS](http://phantomjs.org/), comparing sceenshots is done using
[Blink-diff](http://yahoo.github.io/blink-diff/).
<!-- end -->

![Argus Eyes Example](http://arguseyes.io/static/img/argus-eyes-example.png)


<!-- start -->
## Introduction

After installing, you want to tell argus eyes which pages and which components to check for visual regression.
Argus eyes works by going over a straightforward JSON file containing the resolutions you want to capture, as well as
the pages with their url's, and the components with their CSS-selectors.

Once argus eyes knows where to find the components, it's time for some command line action. The first command you want
to use is `$ argus-eyes capture master`. This makes argus eyes go over the config file and take screenshots of
all specified components in there. The screenshots are saved in **`.argus-eyes/master`** and this set of screenshots can
be compared with another set.

Say you want to compare the components in your dev branch with those in your master branch. Now is the time to switch
branches and do a new `$ argus-eyes capture dev`.

To compare the 2 sets you run `$ argus-eyes compare master dev`, that's it! Argus eyes now checks all supposedly
identical screenshots for visual differences. If differences were found, a new folder **`.argus-eyes/diff_master_dev`**
is created. This folder contains overlay-images of the offending components, highlighting their differences in red.
<!-- end -->


<!-- start -->
## Guide

### Install

- Install [Node.js](http://nodejs.org/), at least v4
- Install argus-eyes using npm:

```bash
$ npm install argus-eyes -g
```


### Step 1: Setup config file

Before argus eyes can measure visual differences, it needs a list of pages and components. By
default argus eyes expects an **`argus-eyes.json`** file in the current working directory.

The config file must be a valid JSON object, containing 3 arrays: **`sizes`**, **`pages`** and
**`components`**. Sizes are simple strings. Page objects require a name, url and list of components. Components require
a name and a CSS-selector. Components can optionally take a list of selectors of elements to ignore, this selector is
appended to the component selector. See the Reference section for a more detailed format description.

**Example config file:**
```json
{
  "sizes": ["320x480", "1280x768"],
  "pages": [ {
      "name": "homepage",
      "url": "http://localhost:3000/",
      "components": [ "navigation", "news-items" ]
    }, {
      "name": "contact",
      "url": "http://localhost:3000/contact.html",
      "components": [ "navigation" ]
    } ],
  "components": [ {
      "name": "navigation",
      "selector": ".nav"
    }, {
      "name": "news-items",
      "selector": ".news-items",
      "ignore": [ ".last-updated" ]
    } ]
}
```


### Step 2: Take screenshots

After setting up the JSON, argus eyes can be put to work. Let's create your first set of screenshots of the components
we specified. To do that, you can use the **`argus-eyes capture`** command:

```bash
$ argus-eyes capture <name>
```

It's best to name your set of screenshots semantically, after your current git branch for example.

Argus eyes now creates a folder called **`.argus-eyes/<name>/`**, and saves the screenshots of all components specified
in the config file.

Now, you can switch branches and save another set of screenshots.


### Step 3: Test for visual differences

When any 2 sets of screenshots were created, argus eyes can compare them for visual differences. Comparison is done with
the **`argus-eyes compare`** command:

```bash
$ argus-eyes compare <left> <right>
```

If visual differences between supposedly identical components are found, a new folder is created. This folder contains
images of the offending components, highlighting their differences in red.

_Note:_ Please make sure to add **`'.argus-eyes'`** to your **`.gitignore`**!

### Gitflow: Testing separate branches

Argus eyes is especially useful for checking visual regression between different branches. Say you're working on a
feature branch and you want to make sure your changes have no unintended side effects:

**On the `feature/navigation` branch:**
```bash
$ argus-eyes capture feature/navigation
```

**On the `develop` branch:**
```bash
$ argus-eyes capture develop
$ argus-eyes compare develop feature/navigation
```

If any differences are found, the visual diff images are stored in **`.argus-eyes/diff_develop_feature-navigation/`**
<!-- end -->


<!-- start -->
## Reference

### Config

Argus eyes will look in the current working directory for a file named **`argus-eyes.json`**. This file contains your
sizes, pages and components. You can specify a different location file using the **`--config`** argument, as described
in [Options](#options).

The config needs to be valid [JSON](http://www.json.org/), and it needs to obey this format specification:

```js
{
  sizes: [
    String               // Size string, example: "1024x768"
    // ...
  ],
  pages: [
    {
      name: String,      // Identifier, used in filenames
      url: String,       // Valid URL
      components: [
        String           // Component identifier
        // ...
      ]
    }
    // ...
  ],
  components: [
    {
      name: String,      // Identifier, used in page objects and filenames
      selector: String,  // CSS selector, to clip the screenshot
      ignore: [          // Optional array of excluded child elements
        String           // CSS selector, to `display:none` a child element
        // ...
      ]
    }
    // ...
  ],
  finished-when: String  // [Optional] Valid JavaScript return statement
}
```

#### Finished when

If provided, the `finished-when` string must contain a return statement that evaluates to `true` whenever the page is
ready to be captured, `false` if it's not yet. If omitted, it defaults to `return true`. Internally, this string is
passed as the only argument to the
[`Function()` function](http://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function),
thus an entire function body is expected and multiple lines are allowed.


### Usage

#### Capture

Run argus eyes and save all the screenshots under **`.argus-eyes/<name>/`**

#### Compare

Compare the two sets of screenshots, creating overlay-images and reporting any difference. The process will exit with
code 0 when no significant differences were found, code 1 when differences were found.

#### Configtest

Run a config file syntax and test. It parses the config file and either reports *Config valid* or detailed information
about the error.


### Options

Argus eyes can take several optional arguments on the CLI. Because `capture` and `compare` take positional arguments, any
of these options must be placed last.

#### Config file

_Default:_ **`argus-eyes.json`**

Use a different config file.

```bash
$ argus-eyes capture feature/navigation --config=config.json
```

#### Threshold

_Default:_ 0

Set the threshold for comparison differences, expects a percentage between 0 and 100. If the difference between 2 files
is bigger than this percentage, it will be treated as different and reported as such.

When comparing screenshots, argus eyes checks if all pixels in screenshots are identical. The difference is calculated
by dividing the number of different pixels by the total number of pixels, giving a percentage. The image is considered
different when this percentage exceeds the threshold percentage.

**Be advised**: You can [exclude html elements](#config) from being captured! You might want to look into that before
increasing the threshold, since that will also increase the chance of unintended changes getting through.

```bash
$ argus-eyes compare develop feature/navigation --threshold=10
```

#### Concurrency

_Default:_ 10

Set the number of PhantomJS instances to run in parallel. This must be a number between 1 and 100. A single PhantomJS
instance is used for every page, not for every component. Screenshots of components are captured synchronously.

**Be advised**: At this moment only the [Capture](#capture) uses this option, since PhantomJS is the biggest performance
hit. The [Compare](#compare) action might be using this in the future as well.

```bash
$ argus-eyes capture feature/navigation --concurrency=25
```

**Be advised**: A higher concurrency isn’t always the better option. Be careful with increasing the concurrency,
PhantomJS takes a lot of memory and it will affect the performance negatively if you run out of memory.

#### Base

_Default:_ **`.argus-eyes`**

Use a different base directory for storing the screenshots and comparison results.

```bash
$ argus-eyes capture develop --base==visual-regression
```

#### Verbose

Turn on verbose output. All output is prefixed with a date string in simplified ISO 8601 format.

```bash
$ argus-eyes compare develop feature/navigation --verbose
```

#### No color

Turn off colored output. Output is colored by default.

```bash
$ argus-eyes capture develop --no-color
```

#### Help

Print usage information.

```bash
$ argus-eyes --help
```

#### Version

Print version.

```bash
$ argus-eyes --version
```
<!-- end -->


## Contributing

Want to contribute to argus-eyes? Awesome! You can contribute in multiple ways. Found a bug? Thought of a new feature?
Check the [contributing guidelines](contributing.md) and find out how to contribute.


## License

Released under the [Creative Commons — Attribution 4.0 International license](https://creativecommons.org/licenses/by/4.0/).
