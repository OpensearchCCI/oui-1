# Consuming OUI

## Requirements and dependencies

OUI expects that you polyfill ES2015 features, e.g. [`babel-polyfill`](https://babeljs.io/docs/usage/polyfill/). Without an ES2015 polyfill your app might throw errors on certain browsers.

OUI also has `moment` and `@elastic/datemath` as dependencies itself. These are already loaded in most Elastic repos, but make sure to install them if you are starting from scratch.

## What's available

OUI publishes React UI components, JavaScript helpers called services, and utilities for writing Jest tests. Please refer to the [OpenSearch UI Framework website](https://elastic.github.io/eui) for comprehensive info on what's available.

OUI is published through [NPM](https://www.npmjs.com/package/@opensearch-project/oui) as a dependency. We also provide a starter projects for:
- [GatsbyJS](https://github.com/elastic/gatsby-oui-starter)
- [NextJS](https://github.com/elastic/next-oui-starter)

### Components

You can import React components from the top-level OUI module.

```js
import {
  OuiButton,
  OuiCallOut,
  OuiPanel,
} from '@opensearch-project/oui';
```

### Services

Most services are published from the `lib/services` directory. Some are published from their module directories in this directory.

```js
import { keys } from '@opensearch-project/oui/lib/services';
import { Timer } from '@opensearch-project/oui/lib/services/time';
```

### Test

Test utilities are published from the `lib/test` directory.

```js
import { findTestSubject } from '@opensearch-project/oui/lib/test';
```

## Using OUI in a standalone project

You can consume OUI in standalone projects, such as plugins and prototypes.

### Importing compiled CSS

Most of the time, you just need the compiled CSS, which provides the styling for the React components.

```js
import '@opensearch-project/oui/dist/oui_theme_light.css';
```

Other compiled themes include:
```js
import '@opensearch-project/oui/dist/oui_theme_dark.css';
```
```js
import '@opensearch-project/oui/dist/oui_theme_cascadia_light.css';
```
```js
import '@opensearch-project/oui/dist/oui_theme_cascadia_dark.css';
```

### Using our Sass variables on top of compiled CSS

If you want to build **on top** of the OUI theme by accessing the Sass variables, functions, and mixins, you'll need to import the Sass globals in addition to the compiled CSS mentioned above. This will require `style`, `css`, `postcss`, and `sass` loaders.

First import the correct colors file, followed by the globals file.

```scss
@import '@opensearch-project/oui/src/themes/oui/oui_colors_light.scss';
@import '@opensearch-project/oui/src/themes/oui/oui_globals.scss';
```

For the dark theme, swap the first import for the dark colors file.

```scss
@import '@opensearch-project/oui/src/themes/oui/oui_colors_dark.scss';
@import '@opensearch-project/oui/src/themes/oui/oui_globals.scss';
```

If you want to use the new, but in progress Cascadia theme, you can import it similarly.

```scss
@import '@opensearch-project/oui/src/themes/oui-cascadia/oui_cascadia_colors_light.scss';
@import '@opensearch-project/oui/src/themes/oui-cascadia/oui_cascadia_globals.scss';
```

### Using Sass to customize OUI

OUI's Sass themes are token based, which can be altered to suite your theming needs like changing the primary color. Simply declare your token overrides before importing the whole OUI theme. This will re-compile **all of the OUI components** with your colors.

*Do not use in conjunction with the compiled CSS.*

Here is an example setup.

```scss
// mytheme.scss
$ouiColorPrimary: #7B61FF;

@import '@opensearch-project/oui/src/theme_light.scss';
```

### Fonts

By default, OUI ships with a font stack that includes some outside, open source fonts. If your system is internet available you can include these by adding the following imports to your SCSS/CSS files, otherwise you'll need to bundle the physical fonts in your build. OUI will drop to System Fonts (which you may prefer) in their absence.

```scss
// index.scss
@import url('https://fonts.googleapis.com/css?family=Roboto+Mono:400,400i,700,700i');
@import url('https://rsms.me/inter/inter-ui.css');
```

The Cascadia theme uses the latest version of Inter that can be grabbed from Google Fonts as well.

```scss
// index.scss
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&family=Roboto+Mono:ital,wght@0,400;0,700;1,400;1,700&display=swap');
```

### Reusing the variables in JavaScript

The Sass variables are also made available for consumption as json files. This enables reuse of values in css-in-js systems like [styled-components](https://www.styled-components.com). As the following example shows, it can also make the downstream components theme-aware without much extra effort:

```js
import * as React from 'react';
import * as ReactDOM from 'react-dom';
import styled, { ThemeProvider } from 'styled-components';
import * as ouiVars from '@opensearch-project/oui/dist/oui_theme_light.json';

const CustomComponent = styled.div`
  color: ${props => props.theme.ouiColorPrimary};
  border: ${props => props.theme.ouiBorderThin};
`;

ReactDOM.render(
  <ThemeProvider theme={ouiVars}>
    <CustomComponent>content</CustomComponent>
  </ThemeProvider>
, document.querySelector('#renderTarget'));
```

### "Module build failed" or "Module parse failed: Unexpected token" error

If you get an error when importing a React component, you might need to configure Webpack's `resolve.mainFields` to `['webpack', 'browser', 'main']` to import the components from `lib` instead of `src`. See the [Webpack docs](https://webpack.js.org/configuration/resolve/#resolve-mainfields) for more info.

### Failing icon imports

To reduce OUI's impact to application bundle sizes, the icons are dynamically imported on-demand. This is problematic for some bundlers and/or deployments, so a method exists to preload specific icons an application needs.

```javascript
import { appendIconComponentCache } from '@opensearch-project/oui/es/components/icon/icon';

import { icon as OuiIconArrowDown } from '@opensearch-project/oui/es/components/icon/assets/arrow_down';
import { icon as OuiIconArrowLeft } from '@opensearch-project/oui/es/components/icon/assets/arrow_left';

// One or more icons are passed in as an object of iconKey (string): IconComponent
appendIconComponentCache({
  arrowDown: OuiIconArrowDown,
  arrowLeft: OuiIconArrowLeft,
});
```

## Customizing with `className`

We do not recommend customizing OUI components by applying styles directly to OUI classes, eg. `.ouiButton`. All components allow you to pass a custom `className` prop directly to the component which will then append this to the class list. Utilizing the cascade feature of CSS, you can then customize by overriding styles so long as your styles are imported **after** the OUI import.

```html
<OuiButton className="myCustomClass__button" />

// Renders as:

<button class="ouiButton myCustomClass__button" />
```

## Using the `test-env` build

OUI provides a separate babel-transformed and partially mocked commonjs build for testing environments in consuming projects. The output is identical to that of `lib/`, but has transformed async functions and dynamic import statements, and also applies some useful mocks. This build mainly targets Kibana's Jest environment, but may be helpful for testing environments in other projects.

### Mapping to the `test-env` directory

In Kibana's Jest configuration, the `moduleNameMapper` option is used to resolve standard OUI import statements with `test-env` aliases.

```js
moduleNameMapper: {
  '@opensearch-project/oui$': '<rootDir>/node_modules/@opensearch-project/oui/test-env'
}
```

This eliminates the need to polyfill or transform the OUI build for an environment that otherwise has no need for such processing.

### Mocked component files

Besides babel transforms, the test environment build consumes mocked component files of the type `src/**/[name].testenv.*`. During the build, files of the type `src/**/[name].*` will be replaced by those with the `testenv` namespace. The purpose of this mocking is to further mitigate the impacts of time- and import-dependent rendering, and simplify environment output such as test snapshots. Information on creating mock component files can be found with [testing documentation](testing.md).
