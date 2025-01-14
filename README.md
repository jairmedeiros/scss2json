# scss-json - forked from [scss-to-json](https://github.com/ryanbahniuk/scss-to-json) ![Build](https://github.com/jairmedeiros/scss-json/actions/workflows/build.yml/badge.svg)

> A package to require SCSS variables in JSON format.

This package allows you to use your SCSS variables in your JS code. Specifically, it takes a SCSS variable file (example below) and will parse, run Sass functions, and convert to JSON format.

## Installation

Install via npm or yarn:

```sh
npm install scss-json --save-dev
```

```sh
yarn add scss-json --dev
```

## Known Issues

There are some issues that need to be fixed:

- [No https path in scss](https://github.com/ryanbahniuk/scss-to-json/issues/26).
- [Variable which including '//' will be regarded as comment](https://github.com/ryanbahniuk/scss-to-json/issues/20).
- [Not correct variable names with '\_\_'](https://github.com/ryanbahniuk/scss-to-json/issues/19).
- [Parenthesis not working property](https://github.com/ryanbahniuk/scss-to-json/issues/16).
- **[Support for maps](https://github.com/ryanbahniuk/scss-to-json/issues/15).**
- [Leading comments break compiling](https://github.com/ryanbahniuk/scss-to-json/issues/13).

> For `v1.0.0`, we just update dependencies and merging old [pulls](https://github.com/ryanbahniuk/scss-to-json/pulls). The next phase is to fix the issues above!

## Input and Output

This package requires a SCSS variables file that is isolated by itself with no other SCSS code. If you are working in a front-end framework or library it is likely that your SCSS code is already set up in this manner. For example, this package will work well with a variables.scss file that looks something like this:

```scss
// Font Sizes
$font-size: 14px;
$font-size-large: $font-size * 1.1;

// Colors
$text-color: #666;
$text-color-light: lighten($text-color, 15%);
$border-color: #123 !global; // use for all borders
```

When run on that code above, scss-json will output the below JSON:

```js
{
  '$font-size': '14px',
  '$font-size-large': '15.4px',
  '$text-color': '#666',
  '$text-color-light': '#8c8c8c',
  '$border-color': '#123'
}
```

Note that scss-json will filter out flags (marked with an !) and comments and evaluate Sass functions before it produces the JSON object.

## Using this Package

In your CommonJS JavaScript file, requiring this package will return a function that takes a file path of your SCSS variables file. It also takes an optional options object, which is detailed in the next section.

```js
var scssJson = require("scss-json");
var path = require("path");

var filePath = path.resolve(__dirname, "colors.scss");
var colors = scssJson(filePath);
```

## Options

The second argument of the returned function is an optional options object. Each option is detailed below:

### Dependencies

SCSS variables files sometimes rely on other SCSS variables defined earlier in your import tree. In order to keep these files isolated (and still produce JSON), you can specify an array of files that your given file depends on. For example, below we are trying to convert our color mapping file, but it depends on the actual color definitions which are found in a different file.

```js
var scssJson = require("scss-json");
var path = require("path");

var filePath = path.resolve(__dirname, "color-mapping.scss");
var dependencyPath = path.resolve(__dirname, "colors.scss");
var colors = scssJson(filePath, {
  dependencies: [{ path: dependencyPath }],
});
```

### Scoping

SCSS variable files are able to provide local and global scope with the following method:

```scss
%scoped {
  $font-size: 14px;
  $font-size-large: $font-size * 1.1 !global;
}

html {
  @extend %scoped;
}
```

This will keep `$font-size` scoped locally inside that block, while allowing it to be used to derive global variables marked with the `!global` flag. These variables will be available throughout your SCSS import tree.

If you use this method in your SCSS variables file, you can provide an option to scss-json to output only the global variables to JSON. The option takes the name of the scoping placeholder as a string.

```js
var scssJson = require("scss-json");
var path = require("path");

var filePath = path.resolve(__dirname, "variables.scss");
var colors = scssJson(filePath, {
  scope: "%scoped",
});
```

### Renaming JSON keys

Change the naming scheme of SCSS variables in the JSON output using the `rename` option:

```scss
$first-variable: red;
$second-variable: blue;
```

```js
var scssJson = require("scss-json");
var path = require("path");
var camelCase = require("lodash.camelCase");
var filePath = path.resolve(__dirname, "variables.scss");
var colors = scssJson(filePath, {
  rename: function (name) {
    return camelCase(name.replace("$", ""));
  },
});
```

When run on the code above, scss-json will output the following JSON:

```js
{
  "firstVariable": "red",
  "secondVariable": "blue"
}
```

The value returned by the `rename` function is used as-is, so be sure to return the original name if no changes are required. While this is best used for non-destructive renaming as shown in the above example, in the event that multiple variables are the same after renaming the JSON will include only the last-specified value.

## CLI

You can also use the CLI `scss-json <file>`.

## Contributing

Pull requests are welcome. If you add functionality, then please add unit tests
to cover it.

## License

MIT © Jair Medeiros
