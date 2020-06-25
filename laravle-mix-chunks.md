# Laravel Mix Configuration For Chunks
```javascript
const mix = require('laravel-mix');

/*
 |--------------------------------------------------------------------------
 | Mix Asset Management
 |--------------------------------------------------------------------------
 |
 | Mix provides a clean, fluent API for defining some Webpack build steps
 | for your Laravel application. By default, we are compiling the Sass
 | file for the application as well as bundling up all the JS files.
 |
 */
mix.babelConfig({
    plugins: ['@babel/plugin-syntax-dynamic-import'],
});
mix.setPublicPath('public/');
mix.webpackConfig({
    output: {
        chunkFilename: 'js/[name].js',
        publicPath: '/project_path/public/'
    },
});

mix.js('resources/js/app.js', 'public/js')
    .sass('resources/sass/app.scss', 'public/css')
    .sourceMaps(false, 'hidden-source-map');
    
/**
* To import components use [() => import('component path')] instead of require('component path').default
*/
    
```

