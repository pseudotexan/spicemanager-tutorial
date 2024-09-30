JJs in NGSpice
=====

.. _introduction:

Introduction
------------

One of the major goals I had while developing Spicewrapper was to also figure out how to get Josephson Junction (JJ) simulations working reliably alongside conventional SPICE simulations, with things like transistors, diodes, passives, etc. This involved creating a custom SPICE model for the JJ, with a new instantiation syntax that is more user-friendly and intuitive than WRSpice. 

WRing out the bad habits of WRSpice
----------------

In order to leave WRSpice behind, you have to first acknowledge what's wrong with the usage patterns for WRSpice.

1. Design-oriented component specification

In WRSpice, JJ definitions include all kinds of irrelevant things like the shunt voltage, superconducting gap, area, etc.  In reality, you actually want to define your JJ based on the intended behavior and the process parameters.  Here's how you can specify a single JJ in your netlist in Spicewrapper:

.. code-block:: console

  *necessary to reference the JJ subcircuit file
  .inc jj_rcsj.sub

  *instance of a jj
  x_jj0 9 0 jj_rcsj 
  +beta_c = 0.9495
  +icrit = 100u
  +jc = 12.0499Meg
  +c_density = 5f

The JJ only needs 4 parameters, and none of them is redundant.  In practice, your process is not likely to change a lot, so you can make your own version of the subcircuit in a local folder where you modify the default values of process parameters like `jc` and `c_density`.  This way, you only need to change `beta_c` and `icrit` on a per-device basis.  I do recommend using a local subcircuit folder that is decoupled from the source code of this repo.  You should make many subcircuits of many things.

A little about the syntax: the prefix letter 'x' in SPICE means this is a reference to a subcircuit.  The next few arguments are the nodes to connect your subcircuit to.  Finally, the name of the subcircuit that will be referenced, in this case jj_rcsj.  I would advise organizing input parameters like I have done in this example, where each variable argument is included on a separate line, with a '+' preceding it to indicate the continuation of the previous line.  This effectively constructs a normal call to the subcircuit that looks neater than a fully in-line call.


----------------
