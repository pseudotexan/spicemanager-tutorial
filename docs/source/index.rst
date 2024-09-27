Welcome to Spicewrapper's documentation!
===================================

**Spicewrapper** is a python library containing a variety of convenience functions for NGSpice.  These include:


- Optimization: specify which netlist parameters you want to vary to achieve a given outcome from the simulation.
   - Randomized grid search: Iterate over the shuffled set of parameter combinations, and find the best one.
   - Basinhopping: Use python's basinhopping optimizer to combine a stochastic search with gradient optimization.
   - Outcomes: specify an objective function for the simulation based on any measurable quantity from it. For example, you can specify the functional form of current through an inductor over time, or instruct it to minimize DC source energy consumption.
   - Objective functions: can specify python functions, or a series of time-value pairs and interpolate around them.
   - Waveform construction: includes a simple GUI tool to draw waveforms quickly.
- Sweeps: run large parameter sweeps for your circuit.  
- Simplified results: simulation outcomes are conveniently returned as Dataframes, organized logically and simply.  No more parsing raw files!
- Built-in multiprocessing: Run as many NGSpice instances in parallel as you want!  Or, at least, with as many physical cores as you have. Boosts simulation throughput considerably, especially for large sweeps and optimization runs.
- Flexible directory management: You can tell spicewrapper where your circuit file is, subcircuit files are, etc., and it will manage the 



Getting started
--------
1. Clone the git repo to your computer from http://www.github.com/pseudotexan/spicewrapper
2. Install dependencies: 
   1. tkinter
   2. numpy
   3. pandas
   4. matplotlib
   5. scipy
   6. pyperclip
3. In the spicewrapper folder, go to ``example_scripts/pulse_filter_grid_optimization_script.py``
4. Near the top, modify the ``path_to_spicewrapper`` path to reflect the actual git directory on your computer.  Note that it will have a different structure depending on the OS you're using. This change will make it easier to copy this example file and use it in other project directories unrelated to spicewrapper.

Running a simple example
--------
In this example, we'll run one of the included scripts, ``example_scripts/pulse_filter_grid_optimization_script.py``, to showcase some of the useful features of Spicewrapper.  

This example uses a netlist file, ``example_circuits/pulse_filter.cir``, which implements a simple RLC lowpass filter.  The values of each element are parameterized in the netlist, i.e. ``.param rval = 50``.  Spicewrapper looks for parameter values in the ``.cir`` file (note: it does not look in subcircuits for parameters) and updates their values as needed for sweeps or optimization runs.  Note that it never directly modifies your circuit file; changes are always made to a temporary copy created in the temporary folder of the spicewrapper directory.

We'll break down the important parts of a Spicewrapper script next.

**Paths and files**
.. code-block:: python
   #define the circuit file and subcircuit path
   circuit_filename = 'pulse_filter.cir'
   
   #here, the script directory is the directory of this file
   script_dir = os.path.dirname(os.path.abspath(__file__))
   
   #define the circuit file path, assumed to be in the same directory as this file in this case
   cir_file_path = os.path.join(script_dir, '..', 'example_circuits', circuit_filename)
   
   #define the subcircuit path - this is where all subcircuits are stored
   subcircuit_path = os.path.join(script_dir, '..', 'included_subcircuits')
These lines tell Spicewrapper where to find the various files that are needed to run the simulation: the main netlist file (circuit file or .cir), and the subcircuits directory, where .sub files will be referenced from.  Spicewrapper will modify the netlist so that any subcircuit includes will reference the actual directory.  

**Optional: Parameter Name Extraction**
You can run this line if you want to save a neatly formatted list of parameters to the clipboard.
``spice_utils.extract_and_format_parameters(cir_file_path)``.

This comes in handy for specifying a parameter sweep and saves you the time of manually hunting through the netlist to find the parameters.  The output will look something like this:

.. code-block:: python

    params = {
        'rval': [50.0, 500.0, 8, 'log'],
        'lval': [1e-07, 1e-06, 8, 'log'],
        'cval': [1e-10, 1e-09, 8, 'log']
    }
