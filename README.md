# Inline Source Only for HTML Webpack Plugin

This plugin extends the [html-webpack-plugin](https://github.com/ampedandwired/html-webpack-plugin) by allowing external scripts or stylesheets to be inlined in the HTML template.

## Motivation
This is the result of getting frustrated at the lack of support for inlined scripts that can be inserted in a template file, persisting order. Although solutions existed in the past, many no longer support new webpack versions or simply did not play nice with other tools (such as [storybook](https://github.com/storybookjs/storybook), since it used it's own HtmlWebpackPlugin setup). So out of necessity, I've hacked together a simple plugin that will basically find all script and link tags with the boolean attribute `inline` (and its respective `src` or `href`) in the template html file and copy the source inbetween those tags. Simple enough right?

## Installation
Using npm:
```bash
npm install --save-dev html-webpack-inline-source-only-plugin
```

Using yarn:
```bash
yarn add --dev html-webpack-inline-source-only-plugin
```

## Setup
First, require the plugin in your webpack config:
```js
const HtmlWebpackPlugin = require('html-webpack-plugin');
const HtmlWebpackInlineSourceOnlyPlugin = require('html-webpack-inline-source-only-plugin');
```

Then, add a new plugin instance to the config:
```js
{
    //...
    plugins: [
        // NOTE: HtmlWebpackInlineSourceOnlyPlugin must come AFTER HtmlWebpackPlugin.
        new HtmlWebpackPlugin(),
        new HtmlWebpackInlineSourceOnlyPlugin(HtmlWebpackPlugin),
    ]
    //...
}
```

And that's it!

## Usage
In your html remplate file, just add the `inline` attribute to a `script` or `link` tag, followed after with the respective `src` or `href` attribute. For `link` tags, you also need to set `rel` to "stylesheet", as the HTML standard suggests ;)

Then, when webpack is compiling, the plugin will replace the tags with the inlined source. Also, the plugin does not parse the content; so all comments (or syntax errors) will be preserved. It's just a simple copy paste.

**Example:**

The html template.
```html
<head>
    <title>Example</title>
    <link inline rel="stylesheet" href="./css/some-stylesheet.css">
    <script inline src="./src/first-executed.js"></script>
</head>
<body>
    <script inline src="./src/some-script.js"></script>
    <script inline src="./src/another-script.js"></script>
</body>
```
The other external files.
```css
/** ./css/some-stylesheet.css */
body {
    margin: 0;
}
```
```js
// ./src/first-executed.js
window.alert('Boop!');
```
```js
// ./src/some-script.js
document.querySelector('title').textContent = "I love cheese!";
```
```js
// ./src/another-script.js
document.querySelector('title').textContent = "I love milk!";

// The title will eventually be "I love milk!", not "I love cheese!", because it is ordered last in the template, as expected from the HTML standard execution order.
```

And here is the generated output.
```html
<!-- The html output -->
<head>
    <title>Example</title>
    <style>
        /** ./css/some-stylesheet.css */
        body {
            margin: 0;
        }
    </style>
    <script>
        // ./src/first-executed.js
        window.alert('Boop!');
    </script>
</head>
<body>
    <script>
        // ./src/some-script.js
        document.querySelector('title').textContent = "I love cheese!";
    </script>
    <script>
        // ./src/another-script.js
        document.querySelector('title').textContent = "I love milk!";

        // The title will eventually be "I love milk!", not "I love cheese!", because it is ordered last in the template, as expected from the HTML standard execution order.
    </script>
</body>
<!-- ... -->
```

## Conclusion
And that is all. Thanks for using this plugin.

Happy coding!
