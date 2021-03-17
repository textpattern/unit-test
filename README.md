# unit-test

Unit (tag language) testing environment for Textpattern CMS.

## What this is

A repo comprising:

* A theme made up of test files (each a Form of type 'Section').
* A bunch of well-known resources/content as an .sql file (forthcoming).
* A way of running tests en-masse or per individual file.
* A report on the success/failure of each test as it is compared to expected output.

## How can I get involved?

Send us tests! Any time you use Textpattern's tag language (`<txp:... />`) in your sites, consider sending a test that demonstrates this usage, along with what you expect the output to display, given the content/environment in which it is executed. We'll then add that as a test here, so as we iterate the core code, we can run these tests to verify that we haven't broken anything in the parser.

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

Note that in YAML, **correct spacing (not tabs!) is vital**. There are two (2) spaces before each directive and four (4) spaces before the content. Failure to adhere to these conventions will mean a test will not be executed.

