# CSS Bundling for Rails

Use [Tailwind CSS](https://tailwindcss.com), [Bootstrap](https://getbootstrap.com/), [Bulma](https://bulma.io/), [PostCSS](https://postcss.org), or [Dart Sass](https://sass-lang.com/) to bundle and process your CSS, then deliver it via the asset pipeline in Rails. This gem provides installers to get you going with the bundler of your choice in a new Rails application, and a convention to use `app/assets/builds` to hold your bundled output as artifacts that are not checked into source control (the installer adds this directory to `.gitignore` by default).

You develop using this approach by running the bundler in watch mode in a terminal with `yarn build:css --watch` (and your Rails server in another, if you're not using something like [puma-dev](https://github.com/puma/puma-dev)). You can also use `./bin/dev`, which will start both the Rails server and the CSS build watcher (along with a JS build watcher, if you're also using `jsbundling-rails`).

Whenever the bundler detects changes to any of the stylesheet files in your project, it'll bundle `app/assets/stylesheets/application.[bundler].css` into `app/assets/builds/application.css`. This build output takes over from the regular asset pipeline default file. So you continue to refer to the build output in your layout using the standard asset pipeline approach with `<%= stylesheet_link_tag "application" %>`.

When you deploy your application to production, the `css:build` task attaches to the `assets:precompile` task to run `yarn build:css` to process your stylesheet entrypoint, as it would in development. This output is then picked up by the asset pipeline, digested, and copied into public/assets, as any other asset pipeline file.

This also happens in testing where the bundler attaches to the `test:prepare` task to ensure the stylesheets have been bundled before testing commences. (Note that this currently only applies to rails `test:*` tasks (like `test:all` or `test:controllers`), not "rails test", as that doesn't load `test:prepare`).

If your test framework does not define a `test:prepare` Rake task, ensure that your test framework runs `css:build` to bundle stylesheets before testing commences.

That's it!

You can configure your bundler options in the `build:css` script in `package.json` or via the installer-generated `tailwind.config.js` for Tailwind or `postcss.config.js` for PostCSS.


## Installation

You must already have node and yarn installed on your system. You will also need npx version 7.1.0 or later. Then:

1. Add `cssbundling-rails` to your Gemfile with `gem 'cssbundling-rails'`
2. Run `./bin/bundle install`
3. Run `./bin/rails css:install:[tailwind|bootstrap|bulma|postcss|sass]`

Or, in Rails 7+, you can preconfigure your new application to use a specific bundler with `rails new myapp --css [tailwind|bootstrap|bulma|postcss|sass]`.


## FAQ

### Do I need to run `yarn install` during deployment or CI?

`yarn install` is not automatically run for any environment, so yes, you'll need to run it on your own, just like you already run `bundle install`. Alternatively, you may use the deprecated `yarn:install` task (or define your own) and then make `css:build` depend on it:

```ruby
# lib/tasks/yarn.rake
Rake::Task["css:build"].enhance(["yarn:install"])
```

### How do I import relative CSS files with Tailwind?

If you want to use `@import` statements as part of your Tailwind application.js file, you need to [configure Tailwind to use `postcss` and then `postcss-import`](https://tailwindcss.com/docs/using-with-preprocessors#build-time-imports). But you should also consider simply referring to your other CSS files directly, instead of bundling them all into one big file. It's better for caching, and it's simpler to setup. You can refer to other CSS files by expanding the `stylesheet_link_tag` in `application.html.erb` like so: `<%= stylesheet_link_tag "application", "other", "styles", "data-turbo-track": "reload" %>`.

### How do I avoid SassC::SyntaxError exceptions on existing projects?

Some CSS packages use new CSS features that are not supported by the default SassC rails integration that previous versions of Rails used. If you hit such an incompatibility, it'll likely be in the form of a `SassC::SyntaxError` when running `assets:precompile`. The fix is to `bundle remove sass-rails` (or `sassc-rails`, if you were using that).

## License

CSS Bundling for Rails is released under the [MIT License](https://opensource.org/licenses/MIT).
