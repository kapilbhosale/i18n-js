# I18n.js

[![Build Status](https://travis-ci.org/fnando/i18n-js.svg?branch=master)](https://travis-ci.org/fnando/i18n-js)
[![Code Climate](https://codeclimate.com/github/fnando/i18n-js.png)](https://codeclimate.com/github/fnando/i18n-js)

It's a small library to provide the Rails I18n translations on the JavaScript.

Features:

- Pluralization
- Date/Time localization
- Number localization
- Locale fallback
- Asset pipeline support
- Lots more! :)

## Usage

### Installation

#### Rails app

Add the gem to your Gemfile.

    source "https://rubygems.org"
    gem "rails", "your_rails_version"
    # You only need this RC version constraint during the development of `3.0.0`, once stable version is released you can remove `rc8` suffix
    # `3.0.0.rc8` is the latest version of released RC version when this entry is changed, you might want to change it later
    gem "i18n-js", ">= 3.0.0.rc8" 

#### Rails app with [Asset Pipeline](http://guides.rubyonrails.org/asset_pipeline.html)

If you're using the [asset pipeline](http://guides.rubyonrails.org/asset_pipeline.html),
then you must add the following line to your `app/assets/javascripts/application.js`.

```javascript
//
// This is optional (in case you have `I18n is not defined` error)
// If you want to put this line, you must put it BEFORE `i18n/translations`
//= require i18n
//
// This is a must
//= require i18n/translations
```

#### Rails app without [Asset Pipeline](http://guides.rubyonrails.org/asset_pipeline.html)


First, put this in your `application.html` (layout file).
Then get the JS files following the instructions below.
```erb
<%# This is just an example, you can put `i18n.js` and `translations.js` anywhere you like %>
<%# Unlike the Asset Pipeline example, you need to require both **in order** %>
<%= javascript_include_tag "i18n" %>
<%= javascript_include_tag "translations" %>
```

**There are two ways to get `translations.js`.**

1. This `translations.js` file can be automatically generated by the `I18n::JS::Middleware`.  
  Just add `config.middleware.use I18n::JS::Middleware` to your `config/application.rb` file.  
  Notice: Don't add this middleware if you are using [asset pipeline](http://guides.rubyonrails.org/asset_pipeline.html)!
2. If you can't or prefer not to generate this file,
  you can move the middleware line to your `config/environments/development.rb` file
  and run `rake i18n:js:export` before deploying.
  This will export all translation files, including the custom scopes 
  you may have defined on `config/i18n-js.yml`.  
  If `I18n.available_locales` is set (e.g. in your Rails `config/application.rb` file) 
  then only the specified locales will be exported.  
  Current version of `i18n.js` will also be exported to avoid version mismatching by downloading.

#### Export Configuration (For translations)

Exported translation files generated by `I18n::JS::Middleware` or `rake i18n:js:export` can be customized with config file `config/i18n-js.yml` (use `rails generate i18n:js:config` to create it). You can even get more files generated to different folders and with different translations to best suit your needs. But this does not affect anything if you use Asset Pipeline.

Examples:
```yaml
translations:
- file: 'public/javascripts/path-to-your-messages-file.js'
  only: '*.date.formats'
- file: 'public/javascripts/path-to-your-second-file.js'
  only: ['*.activerecord', '*.admin.*.title']
```

If `only` is omitted all the translations will be saved. Also, make sure you add that initial `*`; it specifies that all languages will be exported. If you want to export only one language, you can do something like this:
```yaml
translations:
- file: 'public/javascripts/en.js'
  only: 'en.*'
- file: 'public/javascripts/pt-BR.js'
  only: 'pt-BR.*'
```

Optionally, you can auto generate a translation file per available locale if you specify the `%{locale}` placeholder.
```yaml
translations:
- file: "public/javascripts/i18n/%{locale}.js"
  only: '*'
- file: "public/javascripts/frontend/i18n/%{locale}.js"
  only: ['frontend', 'users']
```

You can also include ERB in your config file.
```yaml
translations:
<% Widgets.each do |widget| %>
- file: <%= "'#{widget.file}'" %>
  only: <%= "'#{widget.only}'" %>
<% end %>
```

You are able to exclude certain phrases or whole groups of phrases by
specifying the YAML key(s) in the `except` configuration option. The outputted
JS translations file (exported or generated by the middleware) will omit any
keys listed in `except` configuration param:

```yaml
translations:
  - except: ['active_admin', 'ransack']
```


#### Export Configuration (For other things)

- `I18n::JS.config_file_path`
  Expected Type: `String`  
  Default: `config/i18n-js.yml`  
  Behaviour: Try to read the config file from that location  

- `I18n::JS.export_i18n_js_dir_path`
  Expected Type: `String`  
  Default: `public/javascripts`  
  Behaviour:  
  - Any `String`: considered as a relative path for a folder to `Rails.root` and export `i18n.js` to that folder for `rake i18n:js:export`
  - Any non-`String` (`nil`, `false`, `:none`, etc): Disable `i18n.js` exporting

- You may also set `export_i18n_js` in your config file, e.g.:

```yaml
export_i18n_js_: false
# OR
export_i18n_js: "my/path"

translations:
  - ...
```

To find more examples on how to use the configuration file please refer to the tests.

#### Fallbacks

If you specify the `fallbacks` option, you will be able to fill missing translations with those inside fallback locale(s).  
Default value is `true`.

Examples:
```yaml
fallbacks: true

translations:
- file: "public/javascripts/i18n/%{locale}.js"
  only: '*'
```
This will enable merging fallbacks into each file. (set to `false` to disable).
If you use `I18n` with fallbacks, the fallbacks defined there will be used.
Otherwise `I18n.default_locale` will be used.

```yaml
fallbacks: :de

translations:
- file: "public/javascripts/i18n/%{locale}.js"
  only: '*'
```
Here, the specified locale `:de` will be used as fallback for all locales.

```yaml
fallbacks:
  fr: ["de", "en"]
  de: "en"

translations:
- file: "public/javascripts/i18n/%{locale}.js"
  only: '*'
```
Fallbacks defined will be used, if not defined (e.g. `:pl`) `I18n.fallbacks` or `I18n.default_locale` will be used.

```yaml
fallbacks: :default_locale

translations:
- file: "public/javascripts/i18n/%{locale}.js"
  only: '*'
```
Setting the option to `:default_locale` will enforce the fallback to use the `I18n.default_locale`, ignoring `I18n.fallbacks`.

Examples:
```yaml
fallbacks: false

translations:
- file: "public/javascripts/i18n/%{locale}.js"
  only: '*'
```
You must disable this feature by setting the option to `false`.

To find more examples on how to use the configuration file please refer to the tests.


#### Namespace

Setting the `namespace` option will change the namespace of the output Javascript file to something other than `I18n`.
This can be useful in no-conflict scenarios. Example:

```yaml
translations:
- file: "public/javascripts/i18n/translations.js"
  namespace: "MyNamespace"
```

will create:

```
MyNamespace.translations || (MyNamespace.translations = {});
MyNamespace.translations["en"] = { ... }
```


#### Pretty Print

Set the `pretty_print` option if you would like whitespace and indentation in your output file (default: false)

```yaml
translations:
- file: "public/javascripts/i18n/translations.js"
  pretty_print: true
```


#### Vanilla JavaScript

Just add the `i18n.js` file to your page. You'll have to build the translations object
by hand or using your favorite programming language. More info below.


#### Via NPM with webpack and CommonJS

Add the following line to your package.json dependencies (where version is the version you want - n.b. npm install requires it to be the gzipped tarball, see [npm install](https://www.npmjs.org/doc/cli/npm-install.html))
```javascript
"i18n-js": "http://github.com/fnando/i18n-js/archive/v3.0.0.rc8.tar.gz"
```
Run npm install then use via
```javascript
var i18n = require("i18n-js");
```


### Setting up

You **don't** need to set up a thing. The default settings will work just okay. But if you want to split translations into several files or specify specific contexts, you can follow the rest of this setting up section.

Set your locale is easy as
```javascript
I18n.defaultLocale = "pt-BR";
I18n.locale = "pt-BR";
I18n.currentLocale();
// pt-BR
```

**NOTE:** You can now apply your configuration **before I18n** is loaded like this:
```javascript
I18n = {} // You must define this object in top namespace, which should be `window`
I18n.defaultLocale = "pt-BR";
I18n.locale = "pt-BR";

// Load I18n from `i18n.js`, `application.js` or whatever

I18n.currentLocale();
// pt-BR
```

In practice, you'll have something like the following in your `application.html.erb`:

    <script type="text/javascript">
      I18n.defaultLocale = "<%= I18n.default_locale %>";
      I18n.locale = "<%= I18n.locale %>";
    </script>

You can use translate your messages:

    I18n.t("some.scoped.translation");
    // or translate with explicit setting of locale
    I18n.t("some.scoped.translation", {locale: "fr"});

You can also interpolate values:

    I18n.t("hello", {name: "John Doe"});

You can set default values for missing scopes:

    // simple translation
    I18n.t("some.missing.scope", {defaultValue: "A default message"});

    // with interpolation
    I18n.t("noun", {defaultValue: "I'm a {{noun}}", noun: "Mac"});

You can also provide a list of default fallbacks for missing scopes:

    // As a scope
    I18n.t("some.missing.scope", {defaults: [{scope: "some.existing.scope"}]});

    // As a simple translation
    I18n.t("some.missing.scope", {defaults: [{message: "Some message"}]});

    Default values must be provided as an array of hashs where the key is the
    type of translation desired, a `scope` or a `message`. The translation returned
    will be either the first scope recognized, or the first message defined.

    The translation will fallback to the `defaultValue` translation if no scope
    in `defaults` matches and if no default of type `message` is found.

Translation fallback can be enabled by enabling the `I18n.fallbacks` option:

    <script type="text/javascript">
      I18n.fallbacks = true;
    </script>

By default missing translations will first be looked for in less
specific versions of the requested locale and if that fails by taking
them from your `I18n.defaultLocale`.

    // if I18n.defaultLocale = "en" and translation doesn't exist
    // for I18n.locale = "de-DE" this key will be taken from "de" locale scope
    // or, if that also doesn't exist, from "en" locale scope
    I18n.t("some.missing.scope");

Custom fallback rules can also be specified for a particular language. There
are three different ways of doing it so:

    I18n.locales.no = ["nb", "en"];
    I18n.locales.no = "nb";
    I18n.locales.no = function(locale){ return ["nb"]; };

By default a missing translation will be displayed as

    [missing "name of scope" translation]

While you are developing or if you do not want to provide a translation
in the default language you can set

    I18n.missingBehaviour='guess';

this will take the last section of your scope and guess the intended value.
Camel case becomes lower cased text and underscores are replaced with space

    questionnaire.whatIsYourFavorite_ChristmasPresent

becomes "what is your favorite Christmas present"

In order to still detect untranslated strings, you can
i18n.missingTranslationPrefix to something like:

  I18n.missingTranslationPrefix = 'EE: '

And result will be:

    "EE: what is your favorite Christmas present"

This will help you doing automated tests against your localisation assets.

Pluralization is possible as well and by default provides English rules:

    I18n.t("inbox.counting", {count: 10}); // You have 10 messages

The sample above expects the following translation:

    en:
      inbox:
        counting:
          one: You have 1 new message
          other: You have {{count}} new messages
          zero: You have no messages

**NOTE:** Rails I18n recognizes the `zero` option.

If you need special rules just define them for your language, for example Russian, just add a new pluralizer:

    I18n.pluralization["ru"] = function (count) {
      var key = count % 10 == 1 && count % 100 != 11 ? "one" : [2, 3, 4].indexOf(count % 10) >= 0 && [12, 13, 14].indexOf(count % 100) < 0 ? "few" : count % 10 == 0 || [5, 6, 7, 8, 9].indexOf(count % 10) >= 0 || [11, 12, 13, 14].indexOf(count % 100) >= 0 ? "many" : "other";
      return [key];
    };

You can find all rules on <http://unicode.org/repos/cldr-tmp/trunk/diff/supplemental/language_plural_rules.html>.

If you're using the same scope over and over again, you may use the `scope` option.

    var options = {scope: "activerecord.attributes.user"};

    I18n.t("name", options);
    I18n.t("email", options);
    I18n.t("username", options);

You can also provide an array as scope.

    // use the greetings.hello scope
    I18n.t(["greetings", "hello"]);

#### Number formatting

Similar to Rails helpers, you have localized number and currency formatting.

    I18n.l("currency", 1990.99);
    // $1,990.99

    I18n.l("number", 1990.99);
    // 1,990.99

    I18n.l("percentage", 123.45);
    // 123.450%

To have more control over number formatting, you can use the
`I18n.toNumber`, `I18n.toPercentage`, `I18n.toCurrency` and `I18n.toHumanSize`
functions.

    I18n.toNumber(1000);     // 1,000.000
    I18n.toCurrency(1000);   // $1,000.00
    I18n.toPercentage(100);  // 100.000%

The `toNumber` and `toPercentage` functions accept the following options:

- `precision`: defaults to `3`
- `separator`: defaults to `.`
- `delimiter`: defaults to `,`
- `strip_insignificant_zeros`: defaults to `false`

See some number formatting examples:

    I18n.toNumber(1000, {precision: 0});                   // 1,000
    I18n.toNumber(1000, {delimiter: ".", separator: ","}); // 1.000,000
    I18n.toNumber(1000, {delimiter: ".", precision: 0});   // 1.000

The `toCurrency` function accepts the following options:

- `precision`: sets the level of precision
- `separator`: sets the separator between the units
- `delimiter`: sets the thousands delimiter
- `format`: sets the format of the output string
- `unit`: sets the denomination of the currency
- `strip_insignificant_zeros`: defaults to `false`
- `sign_first`: defaults to `true`

You can provide only the options you want to override:

    I18n.toCurrency(1000, {precision: 0}); // $1,000

The `toHumanSize` function accepts the following options:

- `precision`: defaults to `1`
- `separator`: defaults to `.`
- `delimiter`: defaults to `""`
- `strip_insignificant_zeros`: defaults to `false`
- `format`: defaults to `%n%u`

<!---->

    I18n.toHumanSize(1234); // 1KB
    I18n.toHumanSize(1234 * 1024); // 1MB

#### Date formatting

    // accepted formats
    I18n.l("date.formats.short", "2009-09-18");           // yyyy-mm-dd
    I18n.l("time.formats.short", "2009-09-18 23:12:43");  // yyyy-mm-dd hh:mm:ss
    I18n.l("time.formats.short", "2009-11-09T18:10:34");  // JSON format with local Timezone (part of ISO-8601)
    I18n.l("time.formats.short", "2009-11-09T18:10:34Z"); // JSON format in UTC (part of ISO-8601)
    I18n.l("date.formats.short", 1251862029000);          // Epoch time
    I18n.l("date.formats.short", "09/18/2009");           // mm/dd/yyyy
    I18n.l("date.formats.short", (new Date()));           // Date object

You can also add placeholders to the date format:

    I18n.translations["en"] = {
      date: {
        formats: {
          ordinal_day: "%B %{day}"
        }
      }
    }
    I18n.l("date.formats.ordinal_day", "2009-09-18", { day: '18th' }); // Sep 18th

If you prefer, you can use the `I18n.strftime` function to format dates.

    var date = new Date();
    I18n.strftime(date, "%d/%m/%Y");

The accepted formats are:

    %a  - The abbreviated weekday name (Sun)
    %A  - The full weekday name (Sunday)
    %b  - The abbreviated month name (Jan)
    %B  - The full month name (January)
    %d  - Day of the month (01..31)
    %-d - Day of the month (1..31)
    %H  - Hour of the day, 24-hour clock (00..23)
    %-H - Hour of the day, 24-hour clock (0..23)
    %I  - Hour of the day, 12-hour clock (01..12)
    %-I - Hour of the day, 12-hour clock (1..12)
    %m  - Month of the year (01..12)
    %-m - Month of the year (1..12)
    %M  - Minute of the hour (00..59)
    %-M - Minute of the hour (0..59)
    %p  - Meridian indicator (AM  or  PM)
    %S  - Second of the minute (00..60)
    %-S - Second of the minute (0..60)
    %w  - Day of the week (Sunday is 0, 0..6)
    %y  - Year without a century (00..99)
    %-y - Year without a century (0..99)
    %Y  - Year with century
    %z  - Timezone offset (+0545)

Check out `spec/*.spec.js` files for more examples!

## Using I18n.js with other languages (Python, PHP, ...)

The JavaScript library is language agnostic; so you can use it with PHP, Python, [your favorite language here].
The only requirement is that you need to set the `translations` attribute like following:

    I18n.translations = {};

    I18n.translations["en"] = {
      message: "Some special message for you"
    }

    I18n.translations["pt-BR"] = {
      message: "Uma mensagem especial para você"
    }

## Known Issues

### Missing translations in precompiled file(s) after adding any new locale file

Due to the design of `sprockets`:
- `depend_on` only takes file paths, not directory paths
- registered `preprocessors` are only run when fingerprint of any asset file, including `.erb` files, is changed

New locale files won't be picked up unless any existing locale file content is changed.  
You can workaround it manually by running
```bash
$ rake assets:clobber
```
to clear the asset cache.  
**Or**  
Change something in existing locale file.  
**Or**  
Change `config.assets.version`  

**Note:** `rake assets:clobber` will also remove all fingerprinted assets.  
If you are precompiling assets on target machine(s),
old assets might be removed and cannot be served in cached pages.

Please see issue #213 for detail & related discussion.


## Maintainer

- Nando Vieira - <http://nandovieira.com.br>

## Contributing

Once you've made your great commits:

1. [Fork](http://help.github.com/forking/) I18n.js
2. Create a branch with a clear name
3. Make your changes (Please also add/change spec, README and CHANGELOG if applicable)
4. Push changes to the created branch
5. [Create an Pull Request](http://github.com/fnando/i18n-js/pulls)
6. That's it!

Please respect the indentation rules and code style.
And use 2 spaces, not tabs. And don't touch the versioning thing.

## Running tests

You can run I18n tests using Node.js or your browser.

To use Node.js, install the `jasmine-node` library:

    $ npm install jasmine-node

Then execute the following command from the lib's root directory:

    $ npm test

To run using your browser, just open the `spec/js/specs.html` file.

You can run both Ruby and JavaScript specs with `rake spec`.

## License

(The MIT License)

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
