# unit-test

Unit (tag language) testing environment for Textpattern CMS.

## What this is

A repo comprising:

* A theme made up of test files (each a Form of type 'Section').
* A bunch of well-known resources/content as an .sql file (forthcoming).
* A way of running tests en-masse or per individual file.
* A report on the success/failure of each test as it is compared to expected output.

## How can I get involved?

Developers or community members: send us tests! Any time you use Textpattern's tag language (`<txp:... />`) in your sites or make any changes to the core parser, please create or send a test that demonstrates this usage. All we need is:

* The test itself: tags and surrounding markup, as succinct as possible.
* What you expect the output to display, given the content/environment in which it is executed.
* Information about which particular Textpattern version(s) for which the test is designed.
* Any specific information such as prefs or database content that needs to be in place for the test to succeed.

That test can then be added here, so as we iterate the core code, we can run these tests to verify that we haven't broken anything in the parser.

The more tests, the better.

## Great! How do I send a test?

Either clone this repo and issue a [pull request](https://github.com/textpattern/unit-test/pulls), or simply [raise an issue](https://github.com/textpattern/unit-test/issues) using the provided template as a guide and we'll add your test to the suite.

## What does a test look like?

Tests are in [YAML format](https://phoenixnap.com/blog/what-is-yaml-with-examples), though we only use a very small part of the language. A test comprises three main elements

1. A test name that will be used as a heading. Keep it short and descriptive.
2. The test itself.
3. The expected output of the test.

Here's an example of two tests on the site tags:

~~~ yaml
Site name:
  input: |
    <txp:site_name />
  expect: |
    Unit Test Suite

Site slogan wrapped in paras with class:
  input: |
    <txp:site_slogan wraptag="p" class="slogan" />
  expect: |
    <p class="slogan">Textpattern tag testing</p>
~~~

Input and output can be as simple or complex as you wish, and can span multiple lines because the pipe symbol is used after the input/expect directives.

Note that in YAML, **correct spacing (not tabs!) is vital**. There are two (2) spaces before each directive and four (4) spaces before the content. Failure to adhere to these conventions will mean a test will not be executed and likely show up as a 'warning' (depending on the severity of the malformed test).

To aid with debugging and test creation, you may add a URL parameter `?show=` with any of these four values (comma-separate them if you use more than one):

* `input`: To show the test input read from the YAML doc.
* `expect`: To show the expected output of the test, as read from the YAML doc.
* `output`: To show the actual HTML of the test after parsing.
* `parsed`: To show the actual parsed output, rendered as Textpattern presents it.

By supplying a test without expected output, the test will fail. But if you supply `?show=output` in the URL it will display what the output of the tag is, as HTML. When you’re satisfied it displays as intended, you can copy and paste that output into the `expect: |` rule so it can be used as expected output.

### On-the-fly environment changes

You may alter any of the following items on-the-fly for all tests in a Form:

* Prefs.
* Language strings (provided the language is installed in the database).
* Sections.

Add an `env:` block to your test and then provide data that makes changes to the above items. The changes will remain in force from the test in which they are defined _until the end of the file_ (Form) or until they are subsequently altered. They will automatically be reset to the previous values when the next set of tests in the Form are executed.

Here's an example of altering the language:

~~~ yaml
Language test - English:
  input: |
    <txp:text item="older" wraptag="span" class="string" />
    <p><txp:text item="newer" /></p>
  expect: |
    <span class="string">Older</span>
    <p>Newer</p>

Language test - German:
  input: |
    <txp:text item="older" wraptag="span" class="string" />
    <p><txp:text item="newer" /></p>
  expect: |
    <span class="string">Älter</span>
    <p>Neuer</p>
  env:
    lang: de

Second language test - fails, as German (Vorschau) is still in force:
  input: |
    <span><txp:text item="preview" /></span>
  expect: |
    <span>Preview</span>
~~~

Sections can be altered as follows:

~~~ yaml
Permlink test:
  input: |
    <txp:permlink id="1" />
  expect: |
    <txp:site_url />articles/welcome-to-your-site

Permlink test - YMD:
  input: |
    <txp:permlink id="2" />
  expect: |
    <txp:site_url />articles/hope-for-the-future/meaningful-labor/a-second-exciting-article
  env:
    section:
      articles:
        permlink_mode: breadcrumb_title
~~~

Note that, while adding a new section is supported, its use is limited at present since most tags fetch their content from the database directly each time. Similarly, if you alter/add prefs:

~~~ yaml
  env:
    prefs:
      some_pref: its_value
      permlink_format: 0
~~~

you may find that some tags don't use them, since they get their values from constants, or parts of their output are cached on first use and thus bypasses the pref on subsequent usage.

### Targeting particular versions

Not all tests run on all versions of Textpattern. As things move on, some tests will break, while some will test new features that aren't in older versions. To support this, you can optionally designate input, expect, or entire tests as only for particular versions.

To designate an entire test is only for a particular version, add a `ver:` block to the test:

~~~ yaml
Pagelink root:
  input: |
    <txp:page_url root context="lang" lang="default">
      <txp:permlink />
    </txp:page_url>
  ver: ^4.8.5
~~~

That test will only be run on Textpattern 4.8.5 and higher in the current minor release line. You could also have used >4.8.5, although it is different to the caret in the way it handles higher versions. The caret limits the test to _only_ this 4.8.x branch greater than or equal to 4.8.5, but the _greater than_ will also consider 4.9, 4.10 and so forth.

If instead you wish to run a test for all versions but only have certain parts executed for various versions you can construct sub-entries beneath either `input:` or `expect:` blocks:

~~~ yaml
Pagelink root:
  input: |
    all:
      <txp:page_url type="theme" />
      <txp:page_url type="images_root" />
    ^4.8.5:
      <txp:page_url root context="lang" lang="default">
        <txp:permlink />
      </txp:page_url>
~~~

If you're specifying `input:` tags that differ by version, you should also add corresponding version blocks to your `expect:` results. Note that it's not always the same in reverse: it's perfectly valid for an entire set of tests to run on all versions, but the expected output differs by version. This is usually due to a bug that has been patched in a specific version or a new global attribute affects the tag output in subtle ways.

Versioning follows the same semver process that Composer does, so if you're already familiar with that, it should be second nature. Here's a recap:

* `4.8.0` or `=4.8.0`: Specifically target the named version.
* `4.8.*` : Target any version in the 4.8 family.
* `4.6.0 - 4.7.3` : Target any version in the given range. Note: spaces around the hyphen are vital.
* `^4.9.0-dev` : Anything in the dev codebase matching 4.9.0-dev, or eventual released code in the 4.9.0 family.
* `>4.9.0` : Same as above, but would also consider 4.10.x, 4.11.x and so on.
* `<=4.8.3` : Target versions less than or equal to 4.8.3.

## How do I install this thing?

There are two main routes. One is for adding the framework to your existing site and the other is for installing it with a fresh Textpattern installation.

### To integrate with your existing site

* Download the repo code and extract the zip.
* Copy the entirety of textpattern/setup/themes/tag-test to your themes directory (usually /themes in the root of your site).
* Import the `tag-test` theme from the Presentation>Themes panel.
* Add a section (e.g. 'tests') and assign the tag-test assets: Page: `test-run` / Style: `default`.
* Visit example.org/tests to run the tests in the suite.
* Add your own tests as described above and submit them to us.

### To install from scratch in a new Textpattern

* Clone the repo and merge it with your fresh Textpattern installation.
* Upload everything to your server, including the images and setup directories.
* Install Textpattern as normal via example.org/textpattern/setup. If this is to be nothing but a testing environment, feel free to choose the default public theme as `tag-test`.
* Log into the admin side to complete the installation steps.
* In a section of your choosing (either a new one or an existing one, depending on which theme you chose during installation) assign the tag-test theme’s `test-run` page and `default` stylesheet.
* Visit example.org/tests to run the tests in the suite.
* Add your own tests as described above and submit them to us.
