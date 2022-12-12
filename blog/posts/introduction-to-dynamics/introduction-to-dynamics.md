Introduction to Dynamics
================
Joe(y) Carpinelli
7/4/21

- <a href="#overview" id="toc-overview">Overview</a>
- <a href="#dynamics" id="toc-dynamics">Dynamics</a>
- <a href="#example" id="toc-example">Example</a>
  - <a href="#model-construction" id="toc-model-construction">Model
    Construction</a>
  - <a href="#coding-our-model" id="toc-coding-our-model">Coding our
    Model</a>
  - <a href="#simulating-our-model" id="toc-simulating-our-model">Simulating
    our Model</a>

<div>

> **Offprint**
>
> This is a section from a control theory note-set I’m writing,
> **Exploring Control Theory**. It won’t be done for years! Over time,
> I’ll add chapters to the open-source repository,
> <https://github.com/cadojo/controls>. Some chapters that stand on
> their own will be added as blog posts [here](/blog/)!

</div>

## Overview

Our end goal as controls engineers is to *affect* a system in some
desired way. To do this, we’ll need a mathematical description of our
system. This mathematical description is known as a *model*. There’s a
famous [quote](https://en.wikipedia.org/wiki/All_models_are_wrong) about
mathematical models: “all models are wrong, but some are useful”.

This is really important. We are **always** approximating our system by
describing it with a model. The question we need to ask is “is our
approximate description of our system (our model) good enough?”

<div>

> **Definitions**
>
> - **Model**: a mathematical description of a system

</div>

But what does this model *look like*? To answer this question, let’s
first discuss system dynamics, and equations of motion.

## Dynamics

If we want to *affect* a system, then by definition, the system *should*
be affect-able (is that a word?) by external forces. These “forces”
don’t necessarily need to be physical forces: consider the affect of
introducing new predators in an ecosystem with the goal of affecting
predator-prey populations.

So we have a mathematical description, and we know we’ll need some
changing *parameter* to affect the mathematical description. Sounds like
we need equations! The equations that govern our system are often called
*equations of motion*. Any system that changes due to some external
“force” is known as a *dynamical* system by definition.

The first step to any controls problem is identifying the *dynamics*;
this usually means defining the *equations of motion* for our system. A
set of *equations of motion* which describe our *dynamical* system
**is** our model.

## Example

A really common system in engineering is known as the
**spring-mass-damper**. We can roughly describe this system as a block
on a table, which is connected to a spring. We can *force* (a.k.a.
*affect*) this system by pulling on the block to extend or compress the
spring.

### Model Construction

Let’s figure out our equations of motion. The following question is
usually a useful starting point: “what are the forces on our system?” We
know we’ll have one external force: us pulling or pushing on the block!
Let’s call this external force $f_e$. We’ll also have a force due to the
spring, and a force due to the friction between the block and the table.
The force due to the spring will be *proportional* to the position of
the block with respect to the spring’s neutral position: let’s call the
spring constant $k$. The force due to friction will be proportional to
the *velocity* of the block (let’s forget about static friction for
now): let’s call the coefficient of friction $d$.

With all of the forces identified, we can start constructing the
*eqautions of motion* for this system. If we call our block’s position
$x$, then the acceleration $\ddot{x}$ will be equal to the sum of our
external force $f_e$, the spring force $k x$, and the force due to
friction $d \dot{x}$. Summing these forces produces the following
(common) spring-mass-damper equation, where $m$ is the mass of the
block.

<span id="eq-forced-mass-damper">$$
f_e = m \ddot{x} + d \dot{x} + k x
 \qquad(1)$$</span>

We have our model! This is a second-order differential equation
([Equation 1](#eq-forced-mass-damper)). It helps to divide all variables
into two groups: parameters, and states. States describe the system at
some point in time. In this case, state variables answer the following
questions: “where is the block, what is the velocity of the block, and
what is the acceleration of the block?” One set of valid state variables
for this system is $x$ and $\dot{x}$; we don’t need to include
$\ddot{x}$ in our list of state variables because we can calculate
$\ddot{x}$ from $x$ and $\dot{x}$.

<div>

> **Note**
>
> We say “one set of state variables” because there may (and nearly
> always are) other valid state representations that could completely
> describe this system! State variables are **not** unique.

</div>

If we leave states as symbolic, and then look to the equation(s) of
motion of a system, the *parameters* describe a specific *instance* of a
system. Put another way, no matter what values $f_e$, $d$, and $k$ take,
this equation of motion is identifiable as a spring-mass-damper. For
this reason, we can specify $f_e$, $d$, and $k$ as parameters. We can
assume a unit mass for the block for now.

<div>

> **Note**
>
> The parameter $f_e$ is our *control* parameter in this example,
> because we can change $f_e$ to affect our system. For now, let’s not
> differentiate between control parameters and non-control parameters.

</div>

### Coding our Model

We can simulate these dynamics with Julia’s `DifferentialEquations`
package. If you’re familiar with MATLAB, `DifferentialEquations`
provides numerical integration solvers that are similar to MATLAB’s
`ode45` (and similar) solvers. We can use the `ModelingToolkit` package
to conveniently put our model to code, and interface with
`DifferentialEquations` for simulation.

``` julia
using ModelingToolkit, DifferentialEquations, Plots
```

Now that we have access to `ModelingToolkit`, let’s build the model!

``` julia
@parameters t fₑ d k
@variables x(t) ẋ(t)
δ = Differential(t)

eqs = [
    δ(x) ~ ẋ,
    δ(ẋ)~ - d*ẋ - k*x + fₑ
]

model = ODESystem(eqs, t, [x, ẋ], [fₑ, d, k]; name = :HarmonicOscillator)
```

$$ \begin{align}
\frac{\mathrm{d} x\left( t \right)}{\mathrm{d}t} =& \textnormal{\.{x}}\left( t \right) \\
\frac{\mathrm{d} \textnormal{\.{x}}\left( t \right)}{\mathrm{d}t} =& f_e - k x\left( t \right) - d \textnormal{\.{x}}\left( t \right)
\end{align}
 $$

### Simulating our Model

With `model` defined above, we can use `DifferentialEquations` to
simulate our system. Of course, to do this we’ll need to specify
numerical values for our parameters, and initial conditions (the
simulation starting point) for our state variables. The code below
specifies some arbitrary initial conditions and constant parameter
values, and simulates the resulting dynamics.

``` julia
problem = let x₀ = 0.1, ẋ₀ = 0.0, dₙ = 0.5, kₙ = 0.9, fₙ = 1.0, Δt = 30.0
    ODEProblem(
        model,
        [x => x₀, ẋ => ẋ₀],
        (0.0, Δt),
        [d => dₙ, k => kₙ, fₑ => fₙ]
    )
end

solutions = solve(problem, Tsit5(); reltol = 1e-12, abstol = 1e-12)
plot(solutions; linewidth = 2, dpi = 130, title = "Spring Mass Damper Simulation")
```

![](index_files/figure-commonmark/cell-5-output-1.png)

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
