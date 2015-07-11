---
layout: post
title:  "Welcome to TIL!"
date:   2015-07-07 20:00:00
tags:   update
---
Hello everyone!

This is a new blog I just set up to document and categorize everything I've learned during the day. It won't be just about development though, but I hope this will also be useful to whomever wants to take a look at things.

I'm typing all this before the rest of the default Jekyll post, so I'll just leave it as-is at the end for now.

For today though, I'll just post my current gulpfile. It's what I've come up with, admittedly just to be able to use bower and wiredep with jekyll:

```coffee
{% raw %}
# Gulpfile

gulp        = require 'gulp'
browserSync = require 'browser-sync'
del         = require 'del'
$           = require('gulp-load-plugins')()
wiredep     = require('wiredep').stream
shell       = require 'shelljs'

reload = browserSync.reload

userefOpts =
  jekylljs: (content, target) ->
    "<script src='{{ '/#{target}' | prepend: site.baseurl }}'></script>"
  jekyllcss: (content, target) ->
    "<link rel='stylesheet' href='{{ '/#{target}' |
        prepend: site.baseurl }}'>"

gulp.task 'styles', =>
  # Build scss styles into the staging and tmp directories
  gulp.src 'app/styles/**/*.scss'
    .pipe $.plumber()
    .pipe $.sourcemaps.init()
    .pipe $.sass.sync
      outputStyle: 'expanded'
      precision: 10
      includePaths: ['.']
    .on 'error', $.sass.logError
    .pipe $.autoprefixer browsers: ['last 2 versions']
    .pipe $.sourcemaps.write()
    .pipe gulp.dest '.staging/styles'
    .pipe gulp.dest '.tmp/styles'
    .pipe reload stream: true

gulp.task 'scripts', =>
  # Build coffeescript files into staging and tmp
  gulp.src 'app/scripts/**/*.{coffee,litcoffee}'
    .pipe $.coffee()
    .on 'error', (err) -> console.log 'Error!', err.message
    .pipe gulp.dest '.staging/scripts'
    .pipe gulp.dest '.tmp/scripts'

gulp.task 'jekyll', ['moveAssets'], =>
  # Build the jekyll site for deployment
  # Uses _config.yml only
  # Run after useref to make sure all the files are prepared correctly
  shell.exec 'jekyll build'

gulp.task 'jekyll:tmp', ['moveFiles'], =>
  # Build the jekyll site for serving with livereload
  # Uses both _config.yml and _config.serve.yml
  shell.exec 'jekyll build --config \
        _config.yml,_config.serve.yml'

# These two tasks are to be able to force a reload
gulp.task 'reloadAfterBuild', ['jekyll:tmp'], reload
gulp.task 'reloadAfterScripts', ['scripts'], reload

gulp.task 'moveFiles', ['scripts', 'styles'], =>
  # Moves files to the staging directory to prepare for thejekyll build tasks
  # We're doing this to avoid modifying the files in our working directory,
  # and also because of how jekyll works re: building files and basedir
  files = [
    'app/.no-jekyll'
    'app/*.md'
    'app/*.xml'
    'app/*.html'
    'app/_layouts/**/*'
    'app/_posts/**/*'
    'app/_includes/**/*'
    'app/images/**/*'
    'app/fonts/**/*'
  ]

  gulp.src files, base: './app/'
    .pipe gulp.dest '.staging'

gulp.task 'moveAssets', ['useref'], =>
  # Move compiled, useref'd CSS and JS files to where they belong
  gulp.src '.staging/_includes/scripts/**/*.js'
    .pipe $.rename dirname: 'scripts'
    .pipe gulp.dest '.staging'

  gulp.src '.staging/_includes/styles/**/*.css'
    .pipe $.rename dirname: 'styles'
    .pipe gulp.dest '.staging'

gulp.task 'useref', ['moveFiles'], =>
  # Do the useref on files in the staging directory
  assets = $.useref.assets
    searchPath: ['.', '.staging']

  gulp.src '.staging/_includes/*.html'
    .pipe assets
    .pipe $.if '*.js', $.uglify()
    .pipe $.if '*.css', $.minifyCss compatibility: '*'
    .pipe assets.restore()
    .pipe $.replace /build:(.*?)\ /g, 'build:jekyll$1 '
    .pipe $.useref userefOpts
    .pipe gulp.dest '.staging/_includes'

gulp.task 'images', =>
  # Optimize images
  gulp.src 'app/images/**/*'
    .pipe $.if $.if.isFile, $.cache $.imagemin
      progressive: true
      interlaced: true
      svgoPlugins: [{cleanupIDs: false}]
    .on 'error', (err) ->
      console.log err
      this.end()
    .pipe gulp.dest 'dist/images'

gulp.task 'fonts', =>
  # Copy font files obtained with bower
  bowerFiles = require 'main-bower-files'

  gulp.src bowerFiles({filter: '**/*.{eot,svg,ttf,woff,woff2}'
  }).concat 'app/fonts/**/*'
    .pipe gulp.dest '.tmp/fonts'
    .pipe gulp.dest 'dist/fonts'

gulp.task 'extras', =>
  # Copy all files not handled by jekyll or other gulp tasks
  gulp.src [
    'app/*.*'
    '!app/*.{html,md}'
  ],
    dot: true
  .pipe gulp.dest 'dist'

gulp.task 'clean', =>
  # Clean staging, tmp and _site dirs
  # Only really necessary for build
  del [
    '.staging'
    '.tmp'
    'dist/**/*'
    'dist/.*'
    '!dist/.git'
  ]

# Basic entry point
gulp.task 'build', ['jekyll', 'images', 'fonts', 'extras'], =>
  # Remove stray files from staging, just in case
  del ['.staging/_includes/scripts', '.staging/_includes/styles']

  # Size up the whole thing
  gulp.src 'dist/**/*'
    .pipe $.size {title: 'build', gzip: true}

gulp.task 'serve', ['jekyll:tmp', 'fonts'], =>
  # Run the web server
  # Build a temporary jekyll site out of the staging dir
  # and map the temp dir into the browsersync server
  browserSync
    notify: false
    port: 9000
    server:
      baseDir: ['.tmp', 'app']
      routes:
        '/bower_components': 'bower_components'

  # Trigger a full rebuild when these change
  gulp.watch [
    'app/*.md'
    'app/*.html'
    'app/_includes/**/*'
    'app/_layouts/**/*'
    'app/_posts/**/*'
  ]
  , ['reloadAfterBuild']

  # Trigger just a browser reload
  gulp.watch [
    'app/images/**/*',
    '.tmp/fonts/**/*'
  ]
  .on 'change', reload

  # Path-specific tasks
  gulp.watch 'app/scripts/**/*.{coffee,litcoffee}', ['reloadAfterScripts']
  gulp.watch 'app/styles/**/*.scss', ['styles']
  gulp.watch 'app/fonts/**/*', ['fonts']
  gulp.watch 'bower.json', ['wiredep', 'fonts']

gulp.task 'wiredep', =>
  # Insert bower dependencies into files
  gulp.src '_sass/*.scss'
    .pipe wiredep
      ignorePath: /^(\.\.\/)+/
    .pipe gulp.dest '_sass'

  gulp.src '_includes/*.html'
    .pipe wiredep
      ignorePath: /^(\.\.\/)*\.\./
    .pipe gulp.dest '_includes'

gulp.task 'deploy', ['build'], =>
  # Deploy to Github Pages
  gulp.src 'dist'
    .pipe $.subtree()
    .pipe $.clean()

# Default task
gulp.task 'default', ['build']
{% endraw %}
```

-----

You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve --watch`, which launches a web server and auto-regenerates your site when a file is updated.

To add new posts, simply add a file in the `_posts` directory that follows the convention `YYYY-MM-DD-name-of-post.ext` and includes the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll’s dedicated Help repository][jekyll-help].

[jekyll]:      http://jekyllrb.com
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help
