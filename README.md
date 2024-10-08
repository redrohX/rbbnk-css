# The new CSS framework
This is a proposal to create a fast and simple CSS framework, that can be scaled to use by multiple projects and clients. It should be themable, with enough options and utilities to make things amazing and accessible.

## Set up
The framework prototype is using **Lightning CSS**. An extremely fast CSS parser, transformer, bundler, and minifier. Lightning CSS can bundle imports in a similar way Sass is able to do. But, it also makes the choice on what CSS to process down to code that older browsers understand based on what browsers we want to support. Next to that it also has a built in vendor prefix feature and is great (claims to be the best) at minification.

We can write vanilla CSS, use the latest features like CSS nesting, or use the OKLCH color model, which Lightning CSS will process to something that's supported by the browsers we have defined (by default we target browsers with >= 0.25% market share).

Read the [Lightning CSS documentation](https://lightningcss.dev/docs.html).

### Sass
While my first instinct was to use Sass, I choose not to use it since we're not using any Sass specific features here. A lot of common Sass features have recently become natively available in CSS. While Sass is still great with its mixins and functions, we don't use them. Using Lightning CSS with less additional tooling and less overhead, and working directly with CSS files is the way to go here. If we need the additional features of Sass in the future, then it's easy to port the codebase to Sass without much hassle.

## Installations
Installing the framework should be relatively easy. For now `git clone` and `npm install` in the proper project folder should be enough to get you started. But in the future it should be possible to install it through `npm` or `yarn`, which would then include the framework as a dependency of the project and can then be updated when there is a new version released.

### Build process
In `package.json` there is a basic build command defined (`npm run build`) that generates a minified CSS file that should be used in your project. Additionally in the future it should be possible to `watch` for changes during development, so you don't have to build the CSS manually.

### Global styles
The minified `styles-min.css` file should be included in the base template of the project. Adding it here will make the most relevant CSS globally available for all the pages and components of the project. This includes, amongst other things, basic CSS resets, default design patterns (colors, spacing and typography defaults).

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

### Utility classes
Utility classes are HTML classes that can be used to change a certain styling aspect of an element by adding the class to it. Often used in places where it's not always possible or easy to write custom CSS and ideal for consistency across projects.

There is an optional `utilities-min.css` file that can be included in the same base template as `styles.css`, which makes it possible to use these utility classes on a global level. There are a multitude of utility classes that we can add: spacing, typography, layout, colors, flow, etc.

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

### Utility classes with Shadow DOM
If you use Shadow DOM and want to use the utility classes inside your Web Components, then you should import the `utilities.css` file directly in the Web Component's `<style>` tag. Note that if the stylesheet has been loaded before from other sources (like other Web Components), the browser will **not** download it again, so this won't impact performance negatively.

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
See `examples/page.html` for a real example of a utility class being used in a Custom Element with Shadow DOM.

## Design tokens
The design tokens are defined by a range of custom properties, key-value pairs, in the `config.css` file. It is a collection of attributes that describe any fundamental/atomic visual style. These can be edited and customized based on the branding, design system and/or theme.

### Design token inheritance
For a style to be inherited and applied to an element, it must come from a parent. Any ancestor of Shadow DOM along the DOM tree all the way up to `body` or `html` must have explicitly set an inheritable property. We will have all of the tokens defined on the `:root` element in `config.css`.

For example, we already set the default text color token on the `body` element and since `color` is an inheritable property it applies to all elements on the page, including elements inside the Shadow DOM.

If you're building a Web Component and don't want to use the inherited design token, but another one, you can do the following to overwrite the inherited value. For example if you want to use `--color-text-light--alt` instead of `--color-text-base-light`, do the following inside a `<style>` element in the Web Component Template:

```html
<template>
  <style>
    p {
      color: var(--color-text-light--alt);
    }
  </style>
</template>
```

### Sharing tokens
We want to define tokens that are based on our current branding and assign defaults for text colors, border colors, background colors, fonts etc. We should create default tokens that can easily be configured and used to set-up different themes for different projects. Right now there is a `config.css` file to start, but ideally the tokens should live outside of this framework, so they can be used independently. We can choose to set this up in a JSON file and use a tool like [Style Dictionary](https://amzn.github.io/style-dictionary/#/) to make it possible to use them in different projects such as this, and other applications (Figma, Angular, Vue.js).

Naming tokens is hard and a good naming scheme will make sure we can offer a robust CSS library to everyone. We don't want to get stuck in a place where we run out of t-shirt sizes or have to define ridiculous lengths (lg, xl, xxl, xxxl, xxxxl). The same can be said for the `primary`, `secondary` etc naming schemes. On the other hand, not having to support twenty different spacing sizes makes things a lot easier during implementation and support. If we want to support multiple CSS themes per project, and light and dark schemes, then those factors should be considered when naming the tokens.

### Light and dark schemes
By default light and dark mode schemes are enabled in the `:root` element in `config.css` with `color-scheme: light dark`. The CSS will return one of the two colors options by detecting if the developer has set a light or dark color scheme, or if the user has requested a light or dark color theme on their system.

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

The above can be applied to Web Components, and inside the Shadow DOM. Overwriting light and/or dark schemes can be done in a similar way, by changing the custom properties inside the proper selector of the Web Component.

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

See `examples/page.html` for a real example of this, by playing around with the color mode toggle in the browser developer tools.

### Accessibility
We build websites that are accessible and scalable, and all of our projects should follow the WCAG 2.1 guidelines. Make sure to use proper semantic HTML, WAI-ARIA roles, proper labeling and descriptions and make sure to think about keyboard functionality and contrast where possible.

I've included the screen reader only class: `sr-only`, for content that can be confusing to a user with a screen reader. Elements or actions that would benefit from additional text can use this class to offer context to a screen reader. Below an example where we describe the download icon of an icon button (`aria-label` could also be used here, but is not always included with automated translations).

```html
<button class=”button”>
   <i class=”fa-download” aria-hidden=”true”>
   <span class=”sr-only”>Download file export</span>
</button>
```
