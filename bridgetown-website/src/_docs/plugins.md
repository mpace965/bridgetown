---
order: 210
title: Extend with Plugins
top_section: Configuration
category: plugins
---

Plugins allow you to extend Bridgetown's behavior to fit your needs. You can
write plugins yourself directly in your website codebase, or install gem-based
plugins and [themes](/docs/themes) for a limitless source of new features and
capabilities.

Be sure to
[check out our growing list of official and third-party plugins](/plugins/)
for ways to jazz up your website.

Whenever you need more information about the plugins installed on your site and what they're doing, you can use the `bridgetown plugins list` command. You can also copy content out of gem-based plugins with the `bridgetown plugins cd` command. [Read the command reference for further details.](/docs/commands/plugins)

{%@ Note do %}
  #### Turn Your Plugins into Gems

  If you'd like to maintain plugin separation from your site source code,
  share functionality across multiple projects, and manage dependencies,
  you can create a Ruby gem for private or public distribution. This is also
  how you'd create a [Bridgetown theme](/docs/themes).

  [Read further instructions below on how to create and publish a gem.](#creating-a-gem)
{% end %}

{{ toc }}

## Setup

There are three methods of adding plugins to your site build.

1. Within your site's root folder, there's a `plugins` folder. Write your custom plugins and save them here. Any file ending in `.rb` inside this folder will be loaded automatically before Bridgetown generates your site. Most plugins you write will likely be using the Builder API, so you can add them in `plugins/builders`.

2. Add gem-based plugins to your `Gemfile` by running a command such as:
  ```sh
bundle add bridgetown-feed
  ```
  and then adding an init statement to your `config/initializers.rb` file (such as `init :"bridgetown-feed"`).

3. Run an [automation](/docs/automations) which will install one or more gems along with other set up and configuration:
  ```sh
bin/bridgetown apply https://github.com/bridgetownrb/bridgetown-cloudinary
  ```

{%@ Note type: :warning do %}
  Starting in Bridgetown 1.2, plugins are no longer required to be placed in the `bridgetown_plugins` group for sites which use the new initializers system. Read the [Initializers documentation](/docs/configuration/initializers/) for further details.
{% end %}

## Introduction to the Builder API

The Builder API (with its various <abbr title="Domain-Specific Languages">DSLs</abbr>) is typically the approach you'll use to write Bridgetown plugins.

### Local Custom Plugins

The `SiteBuilder` class in your `plugins` folder provides the a superclass you can inherit from to create a new builder. In `plugins/builders`, you can create one or more subclasses of `SiteBuilder` and write your plugin code within the `build` method which is called automatically by Bridgetown early on in the build process (specifically during the `pre_read` event before content has been loaded from the file system).

```ruby
# plugins/builders/add_some_tags.rb
class Builders::AddSomeTags < SiteBuilder
  def build
    liquid_tag "cool_stuff", :cool_tag
  end

  def cool_tag(attributes, tag)
    "This is so cool!"
  end
end
```

Builders provide a couple of instance methods you can use to reference important data during the build process: `site` and `config`.

So for example you could add data with a generator:

```ruby
class Builders::AddNewData < SiteBuilder
  def build
    generator do
      site.data.new_data = { new: "New stuff" }
    end
  end
end
```

And then reference that data in any template:

```liquid
{% raw %}{{ site.data.new_data.new }}{% endraw %}

   output: New stuff
```

### Gem-based Plugins

For a gem-based plugin, all you have to do is subclass directly from `Bridgetown::Builder`, then define it within your plugin initializer (along with any other configuration set up).

```ruby
# lib/my_nifty_plugin/builder.rb
module MyNiftyPlugin
  class Builder < Bridgetown::Builder
    def build
      this_goes_to = config.my_nifty_plugin.this_goes_to_11
      # do other groovy things
    end
  end
end

# lib/my_nifty_plugin.rb
Bridgetown.initializer :my_nifty_plugin do |config, api_key: ''|
  config.my_nifty_plugin ||= {}
  config.my_nifty_plugin.this_goes_to_11 ||= 11
  config.my_nifty_plugin.api_key = api_key 

  config.builder MyNiftyPlugin::Builder
end
```

Accepting keyword arguments is optional. The above example shows how you can use a keyword parameter to allow users to pass information from their `initializers.rb` file into your plugin. This example allows users to provide a `api_key` parameter from their `initializer.rb` file, and for this example it defaults to an empty string.

Below shows how a user could set the `api_key` parameter from within their `initializers.rb`.
[Refer to the initializers documentation for more about initializers.](/docs/configuration/initializers).

```ruby
# config/initializers.rb
Bridgetown.configure do |config|
  init :my_nifty_plugin do
    api_key "some-api-key"
  end
end

```

[Read further instructions below on how to create and publish a gem.](#creating-a-gem)

## Internal Ruby API

When writing a plugin for Bridgetown, you may sometimes be interacting with
the internal Ruby API. Objects like `Bridgetown::Site`, `Bridgetown::Resource::Base`, `Bridgetown::GeneratedPage`, etc. Other times you may be interacting with Liquid Drops, which are "safe" representations of the internal Ruby API for use in Liquid templates.

Documentation for Bridgetown's class hierarchy is [available on our API website](https://api.bridgetownrb.com).

The simplest way to debug the code you write is to run `bridgetown console` and interact with the API there. You can then copy working code into your plugin, or test out new ideas before committing them to your plugin code. You can also write `binding.irb` at any point in your code, and you'll be dropped into a console when execution pauses at that point.

## Plugin Categories

There are several categories of functionality you can add to your Bridgetown plugin:

### [Helpers](/docs/plugins/helpers)

For Ruby-based templates such as ERB, Serbea, etc., you can provide custom helpers which can be called from your content and design templates.

### [Tags](/docs/plugins/tags)

For Liquid-based templates, you can provide tags (aka "shortcodes") which can be called from your content and design templates. 

### [Filters](/docs/plugins/filters)

You can provide custom Liquid filters to help transform data and content.

### [HTTP Requests and the Resource Builder](/docs/plugins/external-apis)

Easily pull data in from external APIs, and use a special <abbr title="Domain-Specific Language">DSL</abbr> to build resources out of that data.

### [Hooks](/docs/plugins/hooks)

Hooks provide fine-grained control to trigger custom functionality at various points in the build process.

### [HTML & XML Inspectors](/docs/plugins/inspectors)

Post-process the HTML or XML output of resources using the Nokogiri Ruby gem and its DOM-like API.

### [Generators](/docs/plugins/generators)

Generators allow you to automate the creating or updating of content in your site using Bridgetown's internal Ruby APIs.

### [Permalink Placeholders](/docs/plugins/placeholders)

Define lambdas which will be run for any matching placeholders within a permalink.

### [Resource Extensions](/docs/plugins/resource-extensions)

Add new functionality to the resource objects in your site build.

### [Front Matter Loaders](/docs/plugins/front-matter-loaders)

Add new types of front matter to the resource objects and layouts in your site.

### [Commands](/docs/plugins/commands)

Commands extend the `bridgetown` executable using the Thor CLI toolkit.

### [Converters](/docs/plugins/converters)

Converters change a markup language from one format to another.

#### Priority Flag

You can configure a plugin (builders, converters, etc.) with a specific `priority` flag. This flag determines what order the plugin is loaded in.

The default priority is `:normal`. Valid values are:

<code>:lowest</code>, <code>:low</code>, <code>:normal</code>, <code>:high</code>, and <code>:highest</code>.
Highest priority plugins are run first, lowest priority are run last.

Examples of specifying this flag:

```ruby
class Builders::DoImportantStuff < SiteBuilder
  priority :highest

  def build
    # do really important stuff here
  end
end

class Builders::CanWaitUntilLater < SiteBuilder
  priority :low

  def build
    # stuff that'll get run later (after the really important stuff)
  end
end
```

## Cache API

Bridgetown features a [Caching API](/docs/plugins/cache-api) which is used both internally as well as exposed for plugins and components. It can be used to cache the output of deterministic functions to speed up site generation.

## Zeitwerk and Autoloading

Bridgetown uses an autoloading mechanism provided by [Zeitwerk](https://github.com/fxn/zeitwerk), the same code loader used by Rails and many other Ruby-based projects. Zeitwerk uses a specific naming convention so the paths of your Ruby files and the namespaces/modules/classes of your Ruby code are aligned. For example:

```
plugins/my_plugin.rb         -> MyPlugin
plugins/my_plugin/foo.rb     -> MyPlugin::Foo
plugins/my_plugin/bar_baz.rb -> MyPlugin::BarBaz
plugins/my_plugin/woo/zoo.rb -> MyPlugin::Woo::Zoo
```

You can read more about [Zeitwerk's file conventions here](https://github.com/fxn/zeitwerk#file-structure).

In addition to the `plugins` folder provided by default, **you can add your own folders** with autoloading support! Simply add to the `autoload_paths` setting in your config YAML:

```yaml
autoload_paths:
  - loadme
```

Now any Ruby file in your project's `./loadme` folder will be autoloaded. By default, files in your custom folders not "eager loaded", meaning that the Ruby code isn't actually processed unless/until you access the class or module name of the file somewhere in your code elsewhere. This can improve performance in certain cases. However, if you need to rely on the fact that your Ruby code is always loaded when the site is instantiated, simply set `eager` to true in your config:

```yaml
autoload_paths:
  - path: loadme
    eager: true
```

There may be times when you want to bypass Zeitwerk's default folder-based namespacing. For example, if you wanted something like this:

```
plugins/builders/tags.rb   -> Builders::Tags
plugins/helpers/hashify.rb -> Hashify
```

where the files in `builders` use a `Builders` namespace, but the files in `helpers` don't use a `Helpers` namespace, you can use the `autoloader_collapsed_paths` setting:

```yaml
autoloader_collapsed_paths:
  - plugins/helpers
```

And if you don't want namespacing for _any_ subfolders, you can use a glob pattern:

```yaml
autoloader_collapsed_paths:
  - top_level/*
```

Thus no files directly in `top_level` as well as any of its immediate subfolders will be namespaced (that is, no `TopLevel` module will be implied).

## Creating a Gem

The `bridgetown plugins new NAME` command will create an entire gem scaffold
for you to customize and publish to the [RubyGems.org](https://rubygems.org)
and [NPM](https://www.npmjs.com) registries. This is a great way to provide
[themes](/docs/themes), builders, and other sorts of add-on functionality to
Bridgetown websites. You'll want to make sure you update the `gemspec`,
`package.json`, `README.md`, and `CHANGELOG.md` files as you work on your
plugin to ensure all the necessary metadata and user documentation is present
and accounted for.

{%@ Note do %}
  Starting with Bridgetown 1.2, it's a preferred convention to use underscores for your plugin name, aka `my_plugin` rather than `my-plugin`. Many existing plugins start with a `bridgetown` prefix (such as `bridgetown-seo-tag`), but going forward we recommend that if you choose that prefix you still use underscores (aka `bridgetown_plugin_name_here`). While arguably that doesn't fit neatly with standard gem naming conventions, it solves a number of DX headaches. Which is a good thing!
{% end %}

Bridgetown plugins should provide an [initializer](/docs/configuration/initializers) so that they can be easily required and configured via the user's configuration block within `config/initializers.rb`. It's a good practice to ensure at least simple configuration options can alternatively be provided using YAML in `bridgetown.config.yml`.

Make sure you [follow these instructions](/docs/plugins/gems-and-frontend/) to integrate your plugin's frontend code with the users' esbuild or Webpack setup. Also read up on [Source Manifests](/docs/plugins/source-manifests/) if you have layouts, components, resources, static files, and other content you would like your plugin to provide.

You can also provide an automation via your plugin's GitHub repository by adding
`bridgetown.automation.rb` to the root of your repo. This is a great way to
provide advanced and interactive setup for your plugin. [More information on
automations here.](/docs/automations)

When you're ready, publish your plugin gem to the [RubyGems.org](https://rubyplugins.org)
and [NPM](https://www.npmjs.com) registries. There are instructions on how to
do so in the sample README that is present in your new plugin folder under the
heading **Releasing**. Of course you will also need to make sure you've uploaded
your plugin to [GitHub](https://github.com) so it can be included in our
[Plugin Directory](/plugins/) and discovered by Bridgetown site owners far and
wide. Plus it's a great way to solicit feedback and improvements in the form
of open source code collaboration and discussion.

As always, if you have any questions or need support in creating your plugin,
[check out our community resources](/community).

{%@ Note do %}
  #### Testing Your Plugin

  As you author your plugin, you'll need a way to _use_ the gem within a live Bridgetown site. The easiest way to do that is to use a relative local path in the test site's `Gemfile`.

  ```ruby
  gem "my_plugin", :path => "../my_plugin"
  ```

  You would do something similar in your test site's `package.json` as well (be sure to run [yarn link](https://classic.yarnpkg.com/en/docs/cli/link) so Yarn knows not to install your local path into `node_modules`):

  ```json
  "dependencies": {
    "random-js-package": "2.4.6",
    "my_plugin": "../my_plugin"
  }
  ```

  You may need to restart your server at times to pick up changes you make to your plugin (unfortunately hot-reload doesn't always work with gem-based plugins).

  Finally, you should try writing some [tests](http://docs.seattlerb.org/minitest/) in the `test` folder of your plugin. These tests could ensure your content and APIs are working as expected and won't break in the future as code gets updated.
{% end %}
