# The new CSS framework
This is a proposal to create a fast and simple CSS framework, that can be scaled to use by multiple projects and clients. It should be themable, with enough options and utilities to make things amazing and accessible.

## Set up
The framework prototype is using **Lightning CSS**. An extremely fast CSS parser, transformer, bundler, and minifier. I choose not to use the Sass library since we're not using any Sass specific features. A lot of Sass features have recently become available in CSS. Lightning CSS can bundle the imports in a similar way Sass is able to do. But, it also makes the choice on what CSS to process down to code that older browsers understand based on what browsers we want to support. Next to that it also has a built in vendor prefix feature and is great (claims to be the best) at minification. We can write vanilla CSS, use the latest features like CSS nesting, or use the OKLCH color model, which Lightning CSS will process to something that's supported by the browsers we have defined (by default we target browsers with >= 0.25% market share). Sass is still great with its mixins and functions, but in this case I believe it's just not needed and working directly with CSS is the better way to go, with less additional tooling and less overhead.

## Installation
Installing the framework should be relatively easy. For now `git clone` in the proper project folder should be enough to get started. But in the future it should be possible to install it through `npm` or `yarn` which would then include the framework as a dependency of the project and can then be updated when there is a new version released.

### Build process
In `package.json` there is a basic build command defined (`npm run build`) that generates a minified CSS file that should be used in your project. Additionally it should be possible to `watch` for changes, so you don't have to build the CSS manually.

### styles.css
The minified `styles-min.css` file should be included in the base template of the project. Adding it here will make the most relevant CSS globally available for all the pages and components of the project. This includes amongst other things basic CSS resets, default design patterns (colors, spacing and typography defaults).

```html
<!doctype html>
<html lang="nl-NL">
  <head>
    <title>Project title</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <link rel="stylesheet" href="/dist/styles-min.css">
  </head>
  <body>
    ...
  </body>
</html>
```

### utlities.css
There is also a `utilities-min.css` file which can be optionally included in the same base template, depending on if you're using Web Components in the form of Custom Elements with shadow DOM, or need to use utility classes on a more global level.

```html
<!doctype html>
<html lang="nl-NL">
  <head>
    <title>Project title</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <link rel="stylesheet" href="/dist/styles-min.css">
    <!-- Globally available utility classes -->
    <link rel="stylesheet" href="/dist/utilities-min.css">
  </head>
  <body>
    ...
  </body>
</html>
```

If you use shadow DOM and want to use the utility classes in your Web Components, then you should import the file in the Web Component's `<style>` tag directly, so you can use the utility classes inside the shadow DOM of your Web Component. If the stylesheet has been loaded before from other sources, the browser will **not** download it again.

```html
<template>
  <div class="display--flex margin-block--md">
    <h3 class="text-size-md color-secondary">Subscription offer</h3>
    <button type="button" class="button-secondary button-large">Sign up</button>
  </div>
  <style>
    @import "/dist/utilities.min.css";
  </style>
</template>
```

## Design tokens
The design tokens are defined by a range of custom properties. These can be customized and edited by copying and renaming the `config-copy.css` file and then by replacing the empty `config.css` file.

### Inheritance
For a style to be inherited and applied to an element, it must come from a parent. Any ancestor of shadow DOM along the DOM tree all the way up to `body` or `html` must have explicitly set an inheritable property.

For example, we already set the default text color on the body and since `color` is an inheritable property it applies to all elements on the page, including elements inside the shadow DOM.

If you're building a Web Component and not want to use the inherited design token and want to use another one, you can do the following to overwrite the inherited value:

```html
<template>
  <style>
    p {
      color: var(--color-text-light--alt);
    }
  </style>
</template>
```

### Light and dark schemes
By default we will use one color scheme. But it is possible to set up light and dark mode in `config.css`. The CSS will return one of the two colors options by detecting if the developer has set a light or dark color scheme, or if the user has requested a light or dark color theme. Without needing to use the `prefers-color-scheme` media feature query.

```css
/* Assign the custom properties in config.css to the theme colors */
:root {
  color-scheme: light dark;

  --color-text-base-light:  #5645b3;;
  --color-text-base-dark: #c0ffee;
}

/* Custom properties are available in the Web Component's <style> tag  */
p {
  color: light-dark(var(--color-text-base-light), var(--color-text-base-dark));
}
```

The above can be applied to Web Components, and inside the shadow DOM. Overwriting light and/or dark schemes can be done in a similar way, by changing the custom properties inside the proper selector of the Web Component.

```html
<template>
  <section>
    <p>
      CSS is awesome!
    </p>
  </section>
  <style>
    section > p {
      color: light-dark(var(--color-text-base-light--alt), var(--color-text-base-dark--alt));
    }
  </style>
</template>
```
