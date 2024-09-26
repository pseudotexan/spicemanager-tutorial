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
-Flexible directory management: You can tell spicewrapper where your circuit file is, subcircuit files are, etc., and it will manage the 



Getting started
--------
1. Clone the git repo to your computer from http://www.github.com/pseudotexan/spicewrapper
2. In the spicewrapper folder, go to ``example_scripts/pulse_filter_grid_optimization_script.py``
3. Near the top, modify the ``path_to_spicewrapper`` path to reflect the actual git directory on your computer.  This will make it easier to copy this example file and use it in other project directories unrelated to spicewrapper.
4. Run the script 
