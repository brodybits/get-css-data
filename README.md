# get-css-data

[![NPM](https://img.shields.io/npm/v/get-css-data.svg?style=flat-square)](https://www.npmjs.com/package/get-css-data)
[![Build Status](https://img.shields.io/travis/jhildenbiddle/get-css-data/master.svg?style=flat-square)](https://travis-ci.org/jhildenbiddle/get-css-data)
[![Codacy grade](https://img.shields.io/codacy/grade/57eb5b1190054035bbc78ba24868742e.svg?style=flat-square)](https://www.codacy.com/app/jhildenbiddle/get-css-data?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=jhildenbiddle/get-css-data&amp;utm_campaign=Badge_Grade)
[![Codecov](https://img.shields.io/codecov/c/github/jhildenbiddle/get-css-data.svg?style=flat-square)](https://codecov.io/gh/jhildenbiddle/get-css-data)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](https://github.com/jhildenbiddle/get-css-data/blob/master/LICENSE)
[![Tweet](https://img.shields.io/twitter/url/http/shields.io.svg?style=social)](https://twitter.com/intent/tweet?url=https%3A%2F%2Fgithub.com%2Fjhildenbiddle%2Fget-css-data&via=jhildenbiddle&hashtags=css,developers,frontend,javascript)

A micro-library for collecting stylesheet data from link and style nodes.

------

- [Features](#features)
- [Installation](#installation)
- [Example](#example)
- [Options](#options)
- [Contact](#contact)
- [License](#license)

------

## Features

- Collects CSS data from `<link>` and `<style>` nodes
- Collects static [Node.textContent](https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent) or live [CSS Object Model (CSSOM)](https://developer.mozilla.org/en-US/docs/Web/API/CSS_Object_Model) data
- Returns CSS data as a concatenated string and a DOM-ordered array of strings
- Allows document, iframe, and [shadow DOM](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM) traversal
- Handles `@import` rules
- Handles absolute and relative URLs
- Inspect, modify and/or filter CSS data from each node
- Modify XHR object before each request
- UMD and ES6 modules available
- Compatible with modern and legacy browsers (IE9+)
- Lightweight (less than 1.5k min+gzip) and dependency-free

## Installation

NPM:

```shell
npm install get-css-data --save
```

```javascript
// file.js
import getCssData from 'get-css-data';
getCssData({
  // ...
});
```

Git:

```bash
git clone https://github.com/jhildenbiddle/get-css-data.git
```

CDN ([unpkg.com](https://unpkg.com/) shown, also on [jsdelivr.net](https://www.jsdelivr.com/)):

```html
<!-- ES5 in file.html (latest v1.x.x) -->
<script src="https://unpkg.com/get-css-data@1"></script>
<script>
  getCssData({
    // ...
  });
</script>
```

```html
<!-- ES6 module in file.html (latest v1.x.x) -->
<script type="module">
  import getCssData from 'https://unpkg.com/get-css-data@1/dist/get-css-data.esm.min.js';
  getCssData({
    // ...
  });
</script>
```

## Example

HTML:

```html
<!-- file.html -->
<head>
  <link rel="stylesheet" href="style1.css">
  <style>
    @import "style2.css";
    p { color: blue; }
  </style>
</head>
```

CSS:

```css
/* style1.css */
p { color: red; }
```

```css
/* style2.css */
p { color: green; }
```

JavaScript (see [Options](#options) for details)

```javascript
getCss({
  onComplete(cssText, cssArray, nodeArray) {
    console.log(cssText); // 1
    console.log(cssArray); // 2
    console.log(nodeArray); // 3
  }
});

// 1 => 'p { color: red; } p { color: green; } p { color: blue; }'
// 2 => ['p { color: red; }', 'p { color: green; } p { color: blue; }']
// 3 => [<linkElement>, <styleElement>]
```

## Options

- [rootElement](#optionsrootelement)
- [include](#optionsinclude)
- [exclude](#optionsexclude)
- [filter](#optionsfilter)
- [useCSSOM](#optionsusecssom)
- [onBeforeSend](#optionsonbeforesend)
- [onSuccess](#optionsonsuccess)
- [onError](#optionsonerror)
- [onComplete](#optionsoncomplete)

**Example**

```javascript
// Default values shown
getCssData({
  rootElement : document,
  include     : 'link[rel=stylesheet],style',
  exclude     : '',
  filter      : '',
  useCSSOM    : false,
  onBeforeSend(xhr, node, url) {
    // ...
  },
  onSuccess(cssText, node, url) {
    // ...
  },
  onError(xhr, node, url) {
    // ...
  },
  onComplete(cssText, cssArray, nodeArray) {
    // ...
  }
});
```

### options.rootElement

- Type: `object`
- Default: `document`

Root element to traverse for `<link>` and `<style>` nodes.

**Examples**

```javascript
// Document
getCssData({
  rootElement: document // default
});

// Iframe (must be same domain with content loaded)
getCssData({
  rootElement: (myIframe.contentDocument || myIframe.contentWindow.document)
});

// Shadow DOM
getCssData({
  rootElement: myElement.shadowRoot
});
```

### options.include

- Type: `string`
- Default: `"link[rel=stylesheet],style"`

CSS selector matching `<link rel="stylesheet">` and `<style>` nodes to collect data from. The default value includes all style and link nodes.

**Example**

```javascript
getCssData({
  // Include only <link rel="stylesheet"> nodes
  // with an href that does not contains "bootstrap"
  include: 'link[rel=stylesheet]:not([href*=bootstrap])',
});
```

### options.exclude

- Type: `string`

CSS selector matching `<link rel="stylesheet">` and `<style>` nodes to exclude from those matched by [options.include](#optionsinclude).

**Example**

```javascript
getCssData({
  // Of matched `include` nodes, exclude any node
  // with an href that contains "bootstrap"
  exclude: '[href*=bootstrap]',
});
```

### options.filter

- Type: `object`

Regular expression used to filter node CSS data. Each block of CSS data is tested against the filter, and only matching data is processed.

**Example**

```javascript
getCssData({
  // Test each block of CSS for the existence
  // of ".myclass". If found, process the CSS.
  // If not, ignore the CSS.
  filter: /\.myclass/,
});
```

### options.useCSSOM

- Type: `boolean`
- Default: `false`

Determines how CSS data will be collected from `<style>` nodes.

When `false`, static CSS data is collected by reading each node's [textContent](https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent) value. This method is fast, but the data collected will not reflect changes made using the [`deleteRule()`](https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleSheet/deleteRule) or [`insertRule()`](https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleSheet/insertRule) methods. When `true`, live CSS data is collected by iterating over each node's [`CSSRuleList`](https://developer.mozilla.org/en-US/docs/Web/API/CSSRuleList) and concatenating all [`CSSRule.cssText`](https://developer.mozilla.org/en-US/docs/Web/API/CSSRule/cssText) values into a single string. This method is slower, but the data collected accurately reflects all changes made to the stylesheet.

Keep in mind that browsers often drop unrecognized selectors, properties, and values when parsing static CSS. For example, Chrome/Safari will drop declarations with Mozilla's `-moz-` prefix, while Firefox will drop declarations with Chrome/Safari's `-webkit` prefix . This means that data collected when this options is set to `true` will likely vary between browsers and differ from the static CSS collected when it is set to `false`.

**Example**

```javascript
getCssData({
  useCSSOM: false // default
});
```

### options.onBeforeSend

- Type: `function`
- Arguments:
  1. **xhr**: The XHR `object` containing details of the failed request
  1. **node**: The source node `object` reference
  1. **url**: The source URL `string` (`<link>` href or `@import` url)

Callback before each [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) (XHR) is sent. Allows modifying the XML object by setting properties, calling methods, or adding event handlers.

**Example**

```javascript
getCss({
  onBeforeSend(xhr, node, url) {
    // Domain-specific XHR settings
    if (/some-domain.com/.test(url)) {
      xhr.withCredentials = true;
      xhr.setRequestHeader("foo", "1");
      xhr.setRequestHeader("bar", "2");
    }
  }
});
```

### options.onSuccess

- Type: `function`
- Arguments:
  1. **cssText**: A `string` of CSS text from `node` and `url`
  1. **node**: The source node `object` reference
  1. **url**: The source URL `string` (`<link>` href, `@import` url, or page url for `<style>` data)

Callback after CSS data has been collected from each node. Allows modifying the CSS data before it is added to the final output by returning any `string` value (or `false` to skip).

Note that the order in which `<link>` and `@import` CSS data is "successfully" collected (thereby triggering this callback) is not guaranteed as these requests are asynchronous. To access CSS data in DOM order, use the `cssArray` argument passed to the [options.oncomplete](#optionsoncomplete) callback.

**Example**

```javascript
getCss({
  onSuccess(cssText, node, url) {
    // Replace all instances of "color: red" with "color: blue"
    const newCssText = cssText.replace(/color:\s*red\s;/g, 'color: blue;');

    return newCssText;
  }
});
```

### options.onError

- Type: `function`
- Arguments:
  1. **xhr**: The XHR `object` containing details of the failed request
  1. **node**: The source node `object` reference
  1. **url**: The source URL `string` (`<link>` href or `@import` url)

Callback after `<link>` or `@import` request has failed or when
`xhr.responseText` appears to be HTML instead of CSS.

**Example**

```javascript
getCss({
  onError(xhr, node, url) {
    console.log(xhr.status); // 1
    console.log(xhr.statusText); // 2
  }
});

// 1 => '404'
// 2 => 'Not Found'
```

### options.onComplete

- Type: `function`
- Arguments:
  1. **cssText**: A `string` of concatenated CSS text from all nodes in DOM order.
  1. **cssArray**: An `array` of per-node CSS text in DOM order. The node containing each CSS text block is available at the same **nodeArray** index.
  1. **nodeArray**: An `array` of processed `<style>` and `<link>` nodes in DOM order. The CSS text for each node is available at the same **cssArray** index.

Callback after CSS data has been collected from all nodes.

**Example**

```javascript
getCss({
  onComplete(cssText, cssArray, nodeArray) {
    // ...
  }
});
```

## Contact

- Create a [Github issue](https://github.com/jhildenbiddle/get-css-data/issues) for bug reports, feature requests, or questions
- Follow [@jhildenbiddle](https://twitter.com/jhildenbiddle) for announcements
- Add a [star on GitHub](https://github.com/jhildenbiddle/get-css-data) or [tweet](https://twitter.com/intent/tweet?url=https%3A%2F%2Fgithub.com%2Fjhildenbiddle%2Fget-css-data&via=jhildenbiddle&hashtags=css,developers,frontend,javascript) to support the project!

## License

This project is licensed under the MIT License. See the [LICENSE](https://github.com/jhildenbiddle/get-css-data/blob/master/LICENSE) for details.

Copyright (c) 2018 John Hildenbiddle ([@jhildenbiddle](https://twitter.com/jhildenbiddle))
