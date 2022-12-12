Module Hygiene in Python
================
Joe(y) Carpinelli
8/27/22

- <a href="#python-namespaces" id="toc-python-namespaces">Python
  Namespaces</a>
- <a href="#python-modules" id="toc-python-modules">Python Modules</a>
- <a href="#two-categories-of-dependencies"
  id="toc-two-categories-of-dependencies">Two Categories of
  Dependencies</a>
- <a href="#module-example-coordinates.py"
  id="toc-module-example-coordinates.py">Module Example:
  <code>coordinates.py</code></a>
- <a href="#module-cleanup-recipe" id="toc-module-cleanup-recipe">Module
  Cleanup Recipe</a>
  - <a href="#define-exported-names" id="toc-define-exported-names">Define
    Exported Names</a>
  - <a href="#import-temporary-names" id="toc-import-temporary-names">Import
    Temporary Names</a>
  - <a href="#implement-the-public-api"
    id="toc-implement-the-public-api">Implement the Public API</a>
  - <a href="#delete-private-names" id="toc-delete-private-names">Delete
    Private Names</a>
- <a href="#closing" id="toc-closing">Closing</a>

<div>

> **Warning**
>
> This is gonna get pretty opinionated! I want to say this upfront: I’m
> no Python expert. If I got something wrong, please leave a comment,
> send an email, or otherwise let me know!

</div>

## Python Namespaces

Ask yourself the following question: where in a Python program can you
store variables which are distinct from the rest of your program? As you
think of answers to this question, you can loosely think about anything
in Python which is dot-accessible, e.g. `<something>.attribute`; in this
example, `<something>` is functioning as a *namespace*.

<div>

> **Definition: *namespace***
>
> In computer programming, a namespace refers to some element of a
> program which separates named variables from the rest of the program.

</div>

There are plenty of examples of namespaces in Python: modules, classes,
class instances, etc. In fact, all of those examples are `object`
instances in Python! Each assertion made in the code below evaluates to
`True`.

``` python
# Modules are objects!
assert isinstance(__builtins__, object) 

# Classes are objects!
assert isinstance(int, object)

# Class instances are objects!
assert isinstance(int(0), object)
```

For the remainder of this post, let’s focus on Python *modules*
specifically. **We can and should do more to make Python modules
sparse!**

## Python Modules

<div>

> **Definition: *module***
>
> In computer programming, modules are explicit namespace declarations
> which provide separate *compartments* in your program where variable
> names are distinct. A new variable declared in one module is different
> than a new variable declared in another module, even if those
> variables have the same name!

</div>

In Python, modules are defined by `.py` files. There’s no easily
supported way to create modules dynamically, though that would make for
a nice future post! You need to write your module’s contents to a `.py`
file, make that file available on `sys.path`, and then load that module,
i.e. with an `import` statement. The `sys.path` variable is where Python
will look for the name `something` when you type `import something`.
<span class="column-margin">You can check this path yourself! Try
`import sys; print(sys.path)` sometime.</span>

## Two Categories of Dependencies

I’d like to define two terms before we continue: import-time, and
usage-time. Import-time refers to the moment when a module is first
loaded; the module is executed just as if you pasted each line into a
Python interpreter. Usage-time refers to all subsequent usage of a
Python module.

``` python
import numpy # import time
print(numpy.abs(-1)) # usage time
```

This distinction will be important for the module hygiene recipe
described later! Python developers commonly load import-time and
usage-time dependencies at the top of their modules. If you instead
separate import-time and usage-time dependencies, you can safely remove
all import-time dependencies from your module’s namespace!

## Module Example: `coordinates.py`

Let’s pretend the code block below is placed in a file called
`coordinates.py`. If this file is in a directory found in `sys.path`,
then you have access to a module titled `coordinates`! Everything
defined in `coordinates.py` will be available to you, even variables
defined with leading underscores.

In your new module, `coordinates`, we’ll bring in some math functions
from `numpy`, define a public-facing function for users of our module,
and define some temporarily necessary variables. As you read through,
try to find the variables which are only temporarily necessary!

<div>

> **Hint**
>
> Once the full module is loaded, do we really need any of the typing
> variables anymore?

</div>

``` python
"""
Pretend this is the contents of a new Python module, 
"coordinates.py". This module provides common coordinate 
transformations for you!
"""


#
# First, let's define some types to help us 
# write declaritive & modern Python code!
#

Real = int | float 
from typing import NamedTuple


#
# We'll also need some math functions from numpy. 
# We could use Python's built-in `abs`, `sqrt`, 
# `sin`, and `cos` functions, but numpy's versions 
# will have better performance.
#

from numpy import arctan2 as atan2, sqrt, cos, sin


#
# Finally, my favorite Python package: plum-dispatch. 
# It provides Julia-like type dispatching to Python! 
# Of course, you take a performance hit, but look at 
# how clean the code below is! 🤩
#

from plum import dispatch


#
# Now that we have the setup out of the way, let's 
# add some functionality!
#

class Rectangular(NamedTuple):
    """
    Defines a two-dimensional rectangular coordinate.
    """
    x: Real
    y: Real

class Polar(NamedTuple):
    """
    Defines a two-dimensional polar coordinate.
    """
    r: Real
    θ: Real

@dispatch
def rectangular(coordinate: Rectangular) -> Rectangular:
    return coordinate

@dispatch
def rectangular(coordinate: Polar) -> Rectangular:
    return Rectangular(
        x = coordinate.r * cos(coordinate.θ),
        y = coordinate.r * sin(coordinate.θ),
    )

@dispatch
def polar(coordinate: Rectangular) -> Polar:
    return Polar(
        r = sqrt(coordinate.x**2 + coordinate.y**2),
        θ = atan2(-coordinate.y, coordinate.x),
    )

@dispatch
def polar(coordinate: Polar) -> Polar:
    return coordinate
```

Now if you load `coordinates.py` using `import coordinates`, and then
you inspect the contents of the module, what will you find? You’ll
certaintly find names which are the purpose of the package, such as the
`Rectangular` and `Polar`, and the `rectangular` and `polar` methods.
Unfortunately, you’ll also find a lot of names you aren’t intended to
use: `Real`, `NamedTuple`, `atan2`, `sqrt`, `cos`, `sin`, and
`dispatch`.

Most Python developers just ignore all of those names, and that is
absolutely fine. Personally, as I write Python code, these extra names
irk me a bit; why am I providing names which I never intend for my users
to use?

## Module Cleanup Recipe

We can get rid of the extraneous names in our modules by simply calling
`del` on each unwanted name at the end of our module files. If this
pattern is used, users will be able to programatically check for your
public API by reading the contents of `yourmodule.__export__`, and IDE
tab-completion won’t show any private names.

This pattern comes in three steps: define names which you intend to
keep, write the primary contents of your module, and delete all unwanted
names at the end of your module. There’s one additional concept you’ll
need to keep in mind: you can no longer follow the common Python
practice of importing usage-time dependencies at the top-level of your
module! Instead, you can do so at the beginning of each function
definition.

Sound weird? Fear not! The rest of this post walks through this pattern
in detail, and provides a bit more information which can help you
determine if this pattern is useful for you.

### Define Exported Names

Rust specifies elements of a public API by using the keyword `pub`.
Julia specifies exported names with the keyword `export`. Python can
provide the means to accomplish (practically) the same thing! (Let’s
decide to define an `__export__`variable in all of our Python
modules.)<span class="column-margin">This rule also applies to packages
and subpackages because they are also Python modules!</span> This
`__export__` variable should be some kind `str` collection, like a
`list`, `tuple`, `set`, or any other `Iterable` type. Personally, I like
using the `set` type because it feels most in keeping with the *spirit*
of an exported names collection; names can’t be repeated, and order
doesn’t matter!

``` python
__export__ = {
    "Rectangular", "Polar",
    "rectangular", "polar",
}
```

### Import Temporary Names

With this `__export__` collection defined, we can safely include *any*
temporary names we want, just as we normally would. This commonly
includes types defined in the built-in `typing` package. Import all of
the temporary functionality you need, and don’t worry about polluting
your module’s namespace; we’ll clean up this namespace soon!

``` python
Real = int | float
from typing import NamedTuple
from plum import dispatch
```

### Implement the Public API

You have the temporary names you need to add proper typing and
import-time functionality to your public API. Let’s actually write the
API! This is equivalent to all of your exported names, as declared above
in `__export__`.

Note that, so far, we have only imported the import-time dependencies.
We can’t import our usage-time dependencies in our module without adding
them to `__export__`; otherwise, our module will throw a `NameError` as
it attempts to reference previously deleted names!

For example, if we write `from numpy import sin, cos` in our top-level
module, and then delete `sin` and `cos` at the end of the module, all
code which relies on `sin` and `cos` at usage-time will be calling
undefined functions! Rather than throw all of our usage-time
dependencies into `__export__`, we can simply add them to our function
definitions. If you’re worried that those imports will be loaded every
time you call the function, don’t! Each `import` statement within a
function is **only** evaluated **the first time** you call the function.

``` python
class Rectangular(NamedTuple):
    """
    Defines a two-dimensional rectangular coordinate.
    """
    x: Real
    y: Real

class Polar(NamedTuple):
    """
    Defines a two-dimensional polar coordinate.
    """
    r: Real
    θ: Real

@dispatch
def rectangular(coordinate: Rectangular) -> Rectangular:
    return coordinate

@dispatch
def rectangular(coordinate: Polar) -> Rectangular:
    # We've moved this import from the top-level module to within this function! 
    from numpy import sin, cos 
    return Rectangular(
        x = coordinate.r * cos(coordinate.θ),
        y = coordinate.r * sin(coordinate.θ),
    )

@dispatch
def polar(coordinate: Rectangular) -> Polar:
    # We've moved this import from the top-level module to within this function! 
    from numpy import sqrt, arctan2 as atan2
    return Polar(
        r = sqrt(coordinate.x**2 + coordinate.y**2),
        θ = atan2(-coordinate.y, coordinate.x),
    )

@dispatch
def polar(coordinate: Polar) -> Polar:
    return coordinate
```

### Delete Private Names

Now your module definition is coming to a close! You’re done
implementing all of the features of your project, and you’re about to
type `if __name__ == "__main__"`. What I’m proposing, with this whole
blog post, is this: don’t stop there! Add an `else` condition to that
`if` statement!

If your module is not the top-level program (known as “**main**”), then
you should clean up all of your non-exported names with the pattern
below! You need to put this pattern under an `else` condition (or a
`__name__ != "__main__"` condition) because Python interpreters, like
`IPython`, stick “magic” global variables in the top-level namespace.
You don’t want to delete those!

``` python
if __name__ != "__main__":
    for _ in (*locals(), "_"):
        if not _.startswith("__") and _ not in __export__:
            del locals()[_]
```

## Closing

This all might seem a bit strange, but I really like this design
pattern. When writing code in this way, I find I’m constantly thinking
about the required **lifetime** of each name I introduce. For
simplicity’s sake, there’s a strong argument in favor of keeping each
name for the shortest possible lifetime. Following this advice to its
conclusion results in Python modules which are sparse, simple for users
to interact with and understand, and which have clearly separated
import-time and usage-time dependencies.

Check out my open-source Python package,
[`module-hygiene`](https://pypi.org/project/module-hygiene), which
implements the recipe described in this post!

<div class="callout-warning callout callout-style-simple">
    <div class="callout-body d-flex">
        <div class="callout-icon-container">
            <i class="callout-icon"></i>
            </div>
            <div class="callout-body-container">
            <p><i>
                This is personal writing; the words here do not reflect the views of 
                any organization, employer, or entity, except for the author as an 
                individual.
            </i></p>
        </div>
    </div>
</div>

<!-- Begin Mailchimp Signup Form -->
<link href="//cdn-images.mailchimp.com/embedcode/classic-071822.css" rel="stylesheet" type="text/css">
<style type="text/css">
	#mc_embed_signup{background: transparent; clear:left; font:14px Helvetica,Arial,sans-serif; min-width: 240px; max-width: 99%; margin: auto; }
	#mc_embed_signup .mc-field-group input {background-color: transparent; color: inherit;}
	#mc_embed_signup #mc-embedded-subscribe-form div.mce_inline_error {background-color: transparent;}
	/* Add your own Mailchimp form style overrides in your site stylesheet or in this style block.
	   We recommend moving this block and the preceding CSS link to the HEAD of your HTML file. */
</style>
<div id="mc_embed_signup">
<form action="https://dev.us12.list-manage.com/subscribe/post?u=d7ac507b745f56f970d3c5514&amp;id=1b89e6fd67&amp;f_id=00a6bee0f0" method="post" id="mc-embedded-subscribe-form" name="mc-embedded-subscribe-form" class="validate" target="_blank" novalidate>
    <div id="mc_embed_signup_scroll">
	<h2>Subscribe</h2>
	<i>Sign up for monthly updates!</i>
<div class="indicates-required"><span class="asterisk"></span></div>
<div class="mc-field-group">
	<label for="mce-EMAIL">Email Address  <span class="asterisk">*</span>
</label>
	<input type="email" value="" name="EMAIL" class="required email" id="mce-EMAIL">
	<span id="mce-EMAIL-HELPERTEXT" class="helper_text" style="background-color: transparent;"></span>
</div>
<div hidden="true"><input type="hidden" name="tags" value="10469613"></div>
	<div id="mce-responses" class="clear foot">
		<div class="response" id="mce-error-response" style="display:none"></div>
		<div class="response" id="mce-success-response" style="display:none"></div>
	</div>    <!-- real people should not fill this in and expect good things - do not remove this or risk form bot signups-->
    <div style="position: absolute; left: -5000px;" aria-hidden="true"><input type="text" name="b_d7ac507b745f56f970d3c5514_1b89e6fd67" tabindex="-1" value=""></div>
        <div class="optionalParent">
            <div class="clear foot">
                <input type="submit" value="Subscribe" name="subscribe" id="mc-embedded-subscribe" class="button" style="background-color: green;">
            </div>
        </div>
    </div>
</form>
</div>
<script type='text/javascript' src='//s3.amazonaws.com/downloads.mailchimp.com/js/mc-validate.js'></script><script type='text/javascript'>(function($) {window.fnames = new Array(); window.ftypes = new Array();fnames[0]='EMAIL';ftypes[0]='email';fnames[1]='FNAME';ftypes[1]='text';fnames[2]='LNAME';ftypes[2]='text';fnames[3]='ADDRESS';ftypes[3]='address';fnames[4]='PHONE';ftypes[4]='phone';fnames[5]='BIRTHDAY';ftypes[5]='birthday';}(jQuery));var $mcj = jQuery.noConflict(true);</script>
<!--End mc_embed_signup-->
