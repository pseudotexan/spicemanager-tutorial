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
  +c_density = 5f

The prefix letter 'x' in SPICE means this is a reference to a subcircuit.  The next few arguments are the nodes to connect your subcircuit to.  Finally, the name of the subcircuit that will be referenced, in this case jj_rcsj.  You can specify input variables (sort of like parameters) in-line, but I would advise organizing them more like I have done in this example, where each variable argument is included on a separate line, with a '+' preceding it to indicate the continuation of the previous line.  This effectively constructs a normal call to the subcircuit that looks neater.


  
To retrieve a list of random ingredients,
you can use the ``lumache.get_random_ingredients()`` function:

.. autofunction:: lumache.get_random_ingredients

The ``kind`` parameter should be either ``"meat"``, ``"fish"``,
or ``"veggies"``. Otherwise, :py:func:`lumache.get_random_ingredients`
will raise an exception.

.. autoexception:: lumache.InvalidKindError

For example:

>>> import lumache
>>> lumache.get_random_ingredients()
['shells', 'gorgonzola', 'parsley']
