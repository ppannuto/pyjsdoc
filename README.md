Python port of JSDoc.

## Installation ##

The complete script and API is in one Python sourcefile - pyjsdoc.py.  Drop this onto your PYTHONPATH to use as a library, or put/symlink it on your PATH to use as a standalone command-line utility.

Alternatively, this may be installed via "easy_install pyjsdoc".

## Usage ##

The syntax of doc comments and set of supported tags is mostly compatible with the [JSDoc tag reference](http://jsdoc.sourceforge.net/#tagref), but has some differences because PyJSDoc was originally developed with a codebase that used a more functional (vs. OO) style than JSDoc.  It has the following differences:

* @class should be used in a separate doc comment instead of being a tag in the constructor's comment.  Note that the original JSDoc style still works, it'll just pick up the whole constructor's doc comment as class documentation instead of just the @class tag.
* @ignore is supported, but useless.  PyJSDoc determines which functions/methods to document by the presence of doc comments: undocumented functions never show up in the final documentation.
* Instead of the @requires class dependency tag, PyJSDoc provides an @dependency tag in the fileoverview block, which takes a filename (relative to source path).  This performs transitive dependency analysis on all files in the sourcebase, giving you a list of all files that you need to include to use the specified module.
* Instead of separate @type and @return tags, you can format an @return or @returns tag like you would a parameter, as "{Type} Description text", and the parser will pick it up.  You can also use the original JSDoc approach.
* The @fileoverview block supports the additional tags @license and @organization, which take text strings describing the module.

Command-line options are described in the usage text:

    ./pyjsdoc.py --help

Also check out the examples, and build the documentation for them:

    ./pyjsdoc.py -p examples

## Extensibility ##

When used as a library, PyJSDoc is fully extensible without configuration.  Unrecognized tags are accessible as dictionary fields from the appropriate ModuleDoc/FunctionDoc/ClassDoc object.  For example, the following doc comment:

    /**
     * A function with additional info.
     * @function fn_name
     * @my_custom_tag This is custom tag text
     */

Is accessible through the Python API as:

    >>> file['fn_name']['my_custom_tag']
    This is custom tag text.

In PyJSDoc's parent project, we used this in a few places:

* Dependency analysis for automated tests.  We added an @has_test tag to every module with a JSUnit test file, then rebuilt the `<script src=>` tags in the JSUnit file from the all_dependencies module field, so that whenever an upstream dependency changed, we didn't need to manually change every dependent test file.
* GUI generation for front-end JS classes.  In several cases, we had a domain model class that needed to be represented in the UI.  Rather than manually code that logic into the app, we added "@widget {WidgetClass} label Help text" tags to the JSDocs, and then used that to dynamically instantiate UI widgets on the front end.

Also, the -j (--json) command-line switch lets you dump the JSDoc parse tree in JSON format, which lets you use the PyJSDoc front-end from other languages.

## History ##

PyJSDoc started out as a dependency analyzer for home-grown JQuery plugins.  We needed some way to automatically build the dependency chain; the easiest, most maintainable way to do this was through JSDoc comments.  However, JSDoc and JSDoc Toolkit required quite a bit of customization to get at the raw parse tree, and the latter needed a JavaScript runtime on our build box.  It was easier to write a simple regexp-based parser for JSDoc comments and extract only what we needed.

Over time, we found numerous other ways where an extensible, lenient JSDoc parser would be useful in our software.  Our build process and server-side software was all in Python, so extending the existing parsing library was the logical choice.  Eventually, we ended up writing a full documentation generator.

When our startup folded, I ended up with the code, and decided I might as well open-source it.  I ended up rewriting most of it for improved JSDoc compatibility, as well as cleaning up the code, which was initially a quick & dirty hack.

## Credits ##

Written by Jonathan Tang, 2007-2008.
