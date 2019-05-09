# Introduction

`webpack.mix.js`

```
mix.js('resources/assets/js/app.js', 'public/js')
   .sass('resources/assets/sass/app.scss', 'public/css');
```

# Installation & Setup

```
node -v
npm -v
npm install
```

# Running Mix

```
// Run all Mix tasks...
npm run dev

// Run all Mix tasks and minify output...
npm run production
```

## Watching Assets For Changes

```
npm run watch

// OR in some envs

npm run watch-poll
```

# Working With Stylesheets

## Less

The `less` method may be used to compile Less into CSS. Let's compile our primary `app.less` file to `public/css/app.css`.

```
mix.less('resources/assets/less/app.less', 'public/css');
```

Multiple files:

```
mix.less('resources/assets/less/app.less', 'public/css')
   .less('resources/assets/less/admin.less', 'public/css/admin');
```

Customize the file name:

```
mix.less('resources/assets/less/app.less', 'public/stylesheets/styles.css');
```

To override the underlying Less plug-in options, pass an object as the third argument to `mix.less()`:

```
mix.less('resources/assets/less/app.less', 'public/css', {
    strictMath: true
});
```

## Sass

Like LESS ^

Replace `less` with `sass`.

## Stylus

```
mix.stylus('resources/assets/stylus/app.styl', 'public/css');
```

# PostCSS

* Tool for transforming CSS.
* By default, Mix leverages the popular Autoprefixer plug-in to automatically apply all necessary CSS3 vendor prefixes.
* Add any additional plug-ins that are appropriate for application.
* Install the desired plug-in through NPM and then reference it in  `webpack.mix.js` file.

```
mix.sass('resources/assets/sass/app.scss', 'public/css')
   .options({
        postCss: [
            require('postcss-css-variables')()
        ]
   });
```

# Plain CSS

Concatenate some plain CSS stylesheets into a single file.

```
mix.styles([
    'public/css/vendor/normalize.css',
    'public/css/vendor/videojs.css'
], 'public/css/all.css');
```

# URL Processing

* Webpack will rewrite and optimize any url() calls within stylesheets.

```css
.example {
    background: url('../images/example.png');
}
```

By default, Laravel Mix and Webpack will find `example.png`, copy it to `public/images` folder, and then rewrite the `url()` within your generated stylesheet. As such, your compiled CSS will be:

```css
.example {
  background: url(/images/example.png?d41d8cd98f00b204e9800998ecf8427e);
}
```

Absolute paths for any given `url()` will be excluded from URL-rewriting. For example, `url('/images/thing.png')` or `url('http://example.com/images/thing.png')` won't be modified.

To disable this feature:

```
mix.sass('resources/assets/app/app.scss', 'public/css')
   .options({
      processCssUrls: false
   });
```

# Source Maps

* Disabled by default.
* To enable:

```
mix.js('resources/assets/js/app.js', 'public/js')
   .sourceMaps();
```

# Working With JavaScript

```
mix.js('resources/assets/js/app.js', 'public/js');
```

With this single line of code, it takes advantage of:
* ES2015 syntax.
* Modules
* Compilation of .vue files.
* Minification for production environments.

## Vendor Extraction

```
mix.js('resources/assets/js/app.js', 'public/js')
   .extract(['vue'])
```

The `extract` method accepts an array of all libraries or modules to extract into a  `vendor.js` file. Using the above snippet as an example, Mix will generate the following files:
* `public/js/manifest.js`: The Webpack manifest runtime
* `public/js/vendor.js`: Your vendor libraries
* `public/js/app.js`: Your application code

```html
<script src="/js/manifest.js"></script>
<script src="/js/vendor.js"></script>
<script src="/js/app.js"></script>
```

## React

```
mix.react('resources/assets/js/app.jsx', 'public/js');
```

## Vanilla JS

```
mix.scripts([
    'public/js/admin.js',
    'public/js/dashboard.js'
], 'public/js/all.js');
```

A slight variation of `mix.scripts()` is `mix.babel()`. Its method signature is identical to scripts; however, the concatenated file will receive Babel compilation, which translates any ES2015 code to vanilla JavaScript that all browsers will understand.

# Copying Files & Directories

Useful when a particular asset within `node_modules` directory needs to be relocated to  `public` folder.

```
mix.copy('node_modules/foo/bar.css', 'public/css/bar.css');
```

```
mix.copyDirectory('assets/img', 'public/img');
```

# Versioning / Cache Busting

```
mix.js('resources/assets/js/app.js', 'public/js')
   .version();
```

```html
<link rel="stylesheet" href="{{ mix('/css/app.css') }}">
```

Unnecessary in development mode:

```
mix.js('resources/assets/js/app.js', 'public/js');

if (mix.inProduction()) {
    mix.version();
}
```

# Browsersync Reloading

Monitor files for changes, and inject changes into the browser without requiring a manual refresh.

```
mix.browserSync('my-domain.test');

// Or...

// https://browsersync.io/docs/options
mix.browserSync({
    proxy: 'my-domain.test'
});
```

# Environment Variables

Inject environment variables into Mix by prefixing a key in `.env` file with `MIX_`:

`MIX_SENTRY_DSN_PUBLIC=http://example.com`

And use in `webpack.mix.js`

`process.env.MIX_SENTRY_DSN_PUBLIC`

# Notifications

When available, Mix will automatically display OS notifications for each bundle.

To deactivate:

`mix.disableNotifications();`
