JJs in NGSpice
=====

.. _introduction:

Introduction
------------

One of the major goals I had while developing Spicewrapper was to also figure out how to get Josephson Junction (JJ) simulations working reliably alongside conventional SPICE simulations, with things like transistors, diodes, passives, etc. This involved creating a custom SPICE model for the JJ, with a new instantiation syntax that is more user-friendly and intuitive than WRSpice. 

Nothing about this is really specific to Spicewrapper.  It's just meant to make this transition process easier for some people.

WRing out the bad habits of WRSpice
----------------

The following should help you adapt to the different style of using JJs in NGSpice/Spicewrapper.

1. Design-oriented component specification

In WRSpice, JJ definitions include all kinds of irrelevant things like the shunt voltage, superconducting gap, area, etc.  In reality, you actually want to define your JJ based on the intended behavior and the process parameters.  Here's how you can specify a single JJ in your netlist in Spicewrapper:

.. code-block:: console

  *instance of a jj
  x_jj0 9 0 jj_rcsj 
  +beta_c = 0.9495
  +icrit = 100u
  +jc = 12.0499Meg
  +c_density = 5f

The JJ only needs 4 parameters in this subcircuit definition, and none of them is redundant.  In practice, your process is not likely to change a lot, so you can make your own version of the subcircuit in a local folder where you modify the default values of process parameters like ``jc`` and ``c_density``.  This way, you only need to specify ``beta_c`` and ``icrit`` on a per-device basis. I do recommend using a local subcircuit folder that is decoupled from the source code of this repo.  You should make many subcircuits of many things.  To further simplify the instantiation syntax, you can always make a model definition in your circuit file as well, if you want to repeat the same JJ several times.

A little note about the syntax: the prefix letter 'x' in SPICE means this is a reference to a subcircuit.  The next few arguments are the nodes to connect your subcircuit to.  Finally, the name of the subcircuit that will be referenced, in this case jj_rcsj.  I would advise organizing input parameters like I have done in this example, where each variable argument is included on a separate line, with a '+' preceding it to indicate the continuation of the previous line.  This effectively constructs a normal call to the subcircuit that looks neater than a fully in-line call.

Before proceeding, open up ``example_circuits/squid_test.cir`` to see an example circuit using JJs.

2. Using Initial Conditions (IC)

In WRSpice, the typical approach to control initial conditions is to specify a number of voltage-controlled resistors (driven by transient sources) that either block or short-circuit current in different paths at the beginning of the simulation. This is bad for several reasons. It's more computationally intensive, since the value of the resistor is changing over many orders of magnitude over a typically short time.  It also adds unnecessary complexity to both the circuit diagram and the netlist, making it harder to interpret.

You can get rid of these entirely, unless you're using them for the intended purpose, which is a time-dependent physical resistance (like is sometimes used for superconducting nanowire detectors).  Instead, it's preferable to use the initial conditions (IC) feature of SPICE.  This lets you specify a constraint in current or voltage in the circuit at ``t=0``.  

Typically, before the ``.control`` block, you might add a line like 

``.ic v(thisnode) = 0`` 

which specifies that ``thisnode`` has a voltage of 0 at ``t=0``. This is useful for otherwise floating elements that might be ill-defined (like a capacitor on a floating-gate node of a transistor).  Note that you can define several initial conditions at once, and it will attempt to locate a solution that satisfies them all.  If you put in conditions that are impossible to simultaneously achieve, it will obviously lead to problems.

More relevant to superconducting circuits, you can specify currents directly as a constraint in a specific device.  For example,

``L1 4 7 825n IC=10u``

imposes the initial condition that a current of 10 microamperes flows through inductor L1 at ``t=0``. A value of 0 is also acceptable to the program, in case you are wondering.  

Finally, there is one more relevant feature on the netlist that you should pay attention to, when specifying initial conditions: 

``tran 1p 200n 0n 1p UIC``

Note the flag "UIC" at the end in this example.  It means "use initial conditions" and instructs SPICE to skip the initial operating point calculation.  It will just use the ICs that you specify in your netlist.  I have found this crucial for reliable simulation of circuits with JJs.  For almost any other type of circuit, this UIC flag is probably not generally necessary, though it can be handy.  

Using these types of constraints, you can ensure that currents are established in the appropriate paths in your circuit, in a more intuitive and clean manner.  Importantly, if you are importing auto-generated netlists from some other program like LTSpice, make sure that you are not losing track of IC specifications and the UIC flag, if you need to include them in the final netlist with NGSpice.

3. Convergence options

JJs operate in a regime different from most conventional components, with extremely fast switching and strongly voltage-sensitive behavior, on the level of microvolts or smaller.  This means that the convergence and/or tolerance settings for NGSpice are going to be different from usual. I found that default settings for NGSpice sometimes worked, but not reliably enough for practical usage. After some experiments, I arrived at the following settings which seem to work without any issues so far:

.. code-block:: console
 .option abstol=1e-12
 .option reltol=1e-2
 .option vntol=1e-8

This is likely not optimal for performance, so I encourage you to do some convergence settings with these and other options (which you can learn about from the NGSpice manual in more detail).  

4. Further examples
