Adding Block Scope to Python
================
Joe(y) Carpinelli
8/17/22

- <a href="#python-scoping" id="toc-python-scoping">Python Scoping</a>

<div>

> **Caution**
>
> **This post is in-development.** Check back later!

</div>

------------------------------------------------------------------------

## Python Scoping

When you start a Python process, what variables are already defined? If
you start a **standard Python 3.10 interpreter** and call `locals()`,
you’ll see the following output (on MacOs). You’ll also find that
`locals() == globals()` evaluates to `True`! What do the `locals` and
`globals` functions *do*, and how does Python scoping work? Can you use
“block scoping” in Python? All of these questions are answered in this
post!

<div>

> **`python -c 'print(locals())' # on MacOs, with some human formatting for readability`**
>
>     {
>       '__name__': '__main__', 
>       '__doc__': None, 
>       '__package__': None, 
>       '__loader__': <class '_frozen_importlib.BuiltinImporter'>, 
>       '__spec__': None, 
>       '__annotations__': {}, 
>       '__builtins__': <module 'builtins' (built-in)>
>     }

</div>

If you use `IPython`, you’ll find a lot more in your global namespace!
`IPython` has to track your command history *somewhere*, right? It turns
out that your command history, and other `IPython` metadata are all
stored as Python variables in the same Jupyter kernel as your code! Each
of these non-user-facing variables (variables that `IPython` doesn’t
expect you to use) are named with a leading underscore; this is a common
pattern in Python.

``` python
locals()
```

<pre style="white-space:pre;overflow-x:auto;line-height:normal;font-family:Menlo,'DejaVu Sans Mono',consolas,'Courier New',monospace">
<span style="font-weight: bold">{</span>
    <span style="color: #008000; text-decoration-color: #008000">'__name__'</span>: <span style="color: #008000; text-decoration-color: #008000">'__main__'</span>,
    <span style="color: #008000; text-decoration-color: #008000">'__builtin__'</span>: <span style="font-weight: bold">&lt;</span><span style="color: #ff00ff; text-decoration-color: #ff00ff; font-weight: bold">module</span><span style="color: #000000; text-decoration-color: #000000"> </span><span style="color: #008000; text-decoration-color: #008000">'builtins'</span><span style="color: #000000; text-decoration-color: #000000"> </span><span style="color: #000000; text-decoration-color: #000000; font-weight: bold">(</span><span style="color: #000000; text-decoration-color: #000000">built-in</span><span style="color: #000000; text-decoration-color: #000000; font-weight: bold">)</span><span style="font-weight: bold">&gt;</span>,
    <span style="color: #008000; text-decoration-color: #008000">'__builtins__'</span>: <span style="font-weight: bold">&lt;</span><span style="color: #ff00ff; text-decoration-color: #ff00ff; font-weight: bold">module</span><span style="color: #000000; text-decoration-color: #000000"> </span><span style="color: #008000; text-decoration-color: #008000">'builtins'</span><span style="color: #000000; text-decoration-color: #000000"> </span><span style="color: #000000; text-decoration-color: #000000; font-weight: bold">(</span><span style="color: #000000; text-decoration-color: #000000">built-in</span><span style="color: #000000; text-decoration-color: #000000; font-weight: bold">)</span><span style="font-weight: bold">&gt;</span>,
    <span style="color: #008000; text-decoration-color: #008000">'_ih'</span>: <span style="font-weight: bold">[</span>
        <span style="color: #008000; text-decoration-color: #008000">''</span>,
        <span style="color: #008000; text-decoration-color: #008000">'import rich.traceback, rich.pretty\nrich.traceback.install()\nrich.pretty.install()\n\ndel rich'</span>,
        <span style="color: #008000; text-decoration-color: #008000">'locals()'</span>
    <span style="font-weight: bold">]</span>,
    <span style="color: #008000; text-decoration-color: #008000">'_oh'</span>: <span style="font-weight: bold">{}</span>,
    <span style="color: #008000; text-decoration-color: #008000">'_dh'</span>: <span style="font-weight: bold">[</span><span style="color: #800080; text-decoration-color: #800080; font-weight: bold">PosixPath</span><span style="font-weight: bold">(</span><span style="color: #008000; text-decoration-color: #008000">'/Users/joey/GitHub/Personal/aperiodic/blog/posts/python-block-scope'</span><span style="font-weight: bold">)]</span>,
    <span style="color: #008000; text-decoration-color: #008000">'In'</span>: <span style="font-weight: bold">[</span>
        <span style="color: #008000; text-decoration-color: #008000">''</span>,
        <span style="color: #008000; text-decoration-color: #008000">'import rich.traceback, rich.pretty\nrich.traceback.install()\nrich.pretty.install()\n\ndel rich'</span>,
        <span style="color: #008000; text-decoration-color: #008000">'locals()'</span>
    <span style="font-weight: bold">]</span>,
    <span style="color: #008000; text-decoration-color: #008000">'Out'</span>: <span style="font-weight: bold">{}</span>,
    <span style="color: #008000; text-decoration-color: #008000">'get_ipython'</span>: <span style="font-weight: bold">&lt;</span><span style="color: #ff00ff; text-decoration-color: #ff00ff; font-weight: bold">bound</span><span style="color: #000000; text-decoration-color: #000000"> method InteractiveShell.get_ipython of &lt;ipykernel.zmqshell.ZMQInteractiveShell object at </span><span style="color: #008080; text-decoration-color: #008080; font-weight: bold">0x103abc640</span><span style="font-weight: bold">&gt;</span>&gt;,
    <span style="color: #008000; text-decoration-color: #008000">'exit'</span>: <span style="font-weight: bold">&lt;</span><span style="color: #ff00ff; text-decoration-color: #ff00ff; font-weight: bold">IPython.core.autocall.ZMQExitAutocall</span><span style="color: #000000; text-decoration-color: #000000"> object at </span><span style="color: #008080; text-decoration-color: #008080; font-weight: bold">0x103abd3c0</span><span style="font-weight: bold">&gt;</span>,
    <span style="color: #008000; text-decoration-color: #008000">'quit'</span>: <span style="font-weight: bold">&lt;</span><span style="color: #ff00ff; text-decoration-color: #ff00ff; font-weight: bold">IPython.core.autocall.ZMQExitAutocall</span><span style="color: #000000; text-decoration-color: #000000"> object at </span><span style="color: #008080; text-decoration-color: #008080; font-weight: bold">0x103abd3c0</span><span style="font-weight: bold">&gt;</span>,
    <span style="color: #008000; text-decoration-color: #008000">'ojs_define'</span>: <span style="font-weight: bold">&lt;</span><span style="color: #ff00ff; text-decoration-color: #ff00ff; font-weight: bold">function</span><span style="color: #000000; text-decoration-color: #000000"> ojs_define at </span><span style="color: #008080; text-decoration-color: #008080; font-weight: bold">0x103a96d40</span><span style="font-weight: bold">&gt;</span>,
    <span style="color: #008000; text-decoration-color: #008000">'_i'</span>: <span style="color: #008000; text-decoration-color: #008000">'import rich.traceback, rich.pretty\nrich.traceback.install()\nrich.pretty.install()\n\ndel rich'</span>,
    <span style="color: #008000; text-decoration-color: #008000">'_ii'</span>: <span style="color: #008000; text-decoration-color: #008000">''</span>,
    <span style="color: #008000; text-decoration-color: #008000">'_iii'</span>: <span style="color: #008000; text-decoration-color: #008000">''</span>,
    <span style="color: #008000; text-decoration-color: #008000">'_i1'</span>: <span style="color: #008000; text-decoration-color: #008000">'import rich.traceback, rich.pretty\nrich.traceback.install()\nrich.pretty.install()\n\ndel rich'</span>,
    <span style="color: #008000; text-decoration-color: #008000">'_i2'</span>: <span style="color: #008000; text-decoration-color: #008000">'locals()'</span>
<span style="font-weight: bold">}</span>
</pre>

Let’s say you open up a Python
[interpreter](https://docs.python.org/3/tutorial/interpreter.html), also
known as a [REPL](https://en.wikipedia.org/wiki/Read–eval–print_loop),
and define a variable `x`. Then, you write a function, which declares a
local variable `x`. Next, you import `somemodule`, and reference
`somemodule.x`. You now have three separate variables named `x`! When
you continue to write code, which variable will Python pick when you
type the letter `x`? The answer depends on the *context* of the code
you’re writing. This concept in computer science is known as
[*scope*](https://en.wikipedia.org/wiki/Scope_(computer_science)). There
are are elements in a Python program, and practically all popular
programming languages, which serve as proverbial *drawers*, or
*containers*, for your variables. A developer might define a variable
`x` in `somemodule`: that is `somemodule`’s variable `x`! The same
developer, in the same program, might define a local variable `x` in a
function: that is the function’s *local variable* `x`. Finally, the
developer might set a variable `x` at the top-level of the program: that
is the program’s *global variable* `x`.

When a variable is used at some point in a program, Python will first
check for the variable in *local scope*.

And thank goodness! Imagine, for a moment,that Python did **not** check
for the context in which a variable was used. In this alternate
universe, Python simply assumes that each variable name is unique in a
program. Everything would break rather quickly.

You’ve probably heard that [Python doesn’t allow for private
variables](https://docs.python.org/3/faq/programming.html#i-try-to-use-spam-and-i-get-an-error-about-someclassname-spam).[^1]
Using leading underscores in variable names to indicate the *intent* of
private objects is a very common pattern in Python code. I do not like
this! Part of the benefit of

[^1]: Python 3.10
    [documentation](https://docs.python.org/3/faq/programming.html#i-try-to-use-spam-and-i-get-an-error-about-someclassname-spam)
    states: *“…an outside user can still deliberately access the
    `_classname__spam` attribute, and private values are visible in the
    object’s `__dict__`. Many Python programmers never bother to use
    private variable names at all.”*


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
