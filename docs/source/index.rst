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

Prologue: Python and NGSpice
--------

**Windows**

1. Install the official python distributable https://www.python.org/downloads/windows/ 
2. Install NGSpice, latest version: https://ngspice.sourceforge.io/download.html 
3. Move the NGSpice folder wherever you want it, then copy the path to the folder where the .exe is stored (you can click on the windows explorer bar and ctrl+c to grab it once you've navigated to it)
4. Add the NGSpice path to your Windows PATH environment variables:
   1. Press windows and type "Environment variables" and go to the "edit environment variables for your account" suggestion
   2. Click on the "Path" user variable and click the "Edit" button
   3. Click "new" on the popup window, and paste the path that you copied to the ngspice executable earlier.  **Note: the path shouldn't include the .exe name itself!**
   4. Hit OK to close all the windows for this stuff.
5. 

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

The goal of this script is to optimize the L and C values to smooth out a sharp input voltage pulse.  It will search a log-spaced grid of parameter value combinations and find the set of values with the most nearly ideal result.  We'll break down the script and explain how this happens.

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

This comes in handy for specifying a parameter sweep and saves you the time of manually hunting through the netlist to find the parameters.  The clipboard will grab something like this when it's run:

.. code-block:: python

    params = {
        'rval': [50.0, 500.0, 8, 'log'],
        'lval': [1e-07, 1e-06, 8, 'log'],
        'cval': [1e-10, 1e-09, 8, 'log']
    }

**Parameters**

In this example, the parameters are specified like this:

.. code-block:: python

   #define the parameters that we want to sweep during the optimization
   params = {
       'lval': [1e-10, 1e-6, 8,'log'], #parameter name, min, max, number of points, type of sweep ('lin' or 'log')
       'cval': [5e-14, 5e-11, 8,'log'],
   }

The simulation will iterate over all combinations of both variables, in this case, an 8x8 grid, where the values are logarithmically spaced from the minimum to the maximum value for each parameter.  The values are initially shuffled into a random order to avoid "hugging" the edge values at the beginning.

**Objective Waveforms and User Functions**

How does it know what a good simulation result is?  Spicewrapper lets you define a user function like so:

.. code-block:: python

   def user_function(df):
    wf_result = data_processing.evaluate_objective_waveforms(waveforms, df)
    return wf_result

The ``waveforms`` passed to ``evaluate_objective_waveforms`` are specified here as:

.. code-block:: python

   #define the objective waveform that we want the variable to match
   #in this case, we want to smooth out the transitions of the pulse
   objective_waveform1 = {
       'variable': 'v(outpos)', #the name of the output variable to match, such as a voltage at a node or current through a device
       'time_value_pairs': 
           [(0, 0), #time, value pairs
            (1e-9, 0),
            (1.3e-9, 0.2),
            (1.5e-9, 0.8),
            (1.8e-9, 1), 
            (6.0e-9, 1), 
            (6.3e-9, 0.8),
            (6.5e-9, 0.2),
            (6.8e-9, 0),
            (7.8e-9, 0)],
       'deviation_size': 0.01, #actual deviation of the variable that is allowable from its objective value
       'interpolation_method': 'hermite', #how to interpolate between specified time value pairs
       'power': 1 #higher values penalize deviations more heavily
   }
   
   #this is the list of objective waveforms that we want to match during the optimization
   waveforms = [objective_waveform1]

In this simple example, we've written out a small set of discrete values that the variable ``v(outpos)`` (voltage at node "outpos") should closely follow over time.  We can specify other things about the penalty for deviations in the waveform as well.  You can include any number of waveforms to evaluate, or none at all.  Each time an NGSpice simulation completes, it evaluates the specified variable and compares the result to the "desired waveform" that you specified for it.  By default, Spicewrapper uses a convenient heuristic we call "deviational loss."  In short, the absolute error between the desired and actual values is taken as a fraction of a "deviation_size" and raised to a penalty power.  Note that the scale of deviation size is absolute, not fractional.  This has some advantages over a simple RMSE evaluation in that it may be less biased for functions with wide extremes in values.  Nevertheless, you may wish to use your own metric, and in that case you can define your ``user_function`` any way you want.  It just has to take in a ``spice_df`` dataframe (see formatting notes below) and return a scalar score value.

**Running the sweep**

Next, we call ``run_spicemanager`` to begin the optimization process.

.. code-block:: python

   best_result,all_results = simulation_runner.run_spicemanager(
    cir_file_path,
    subcircuit_path,
    params,
    user_function,
    process_timeout = 60, #timeout for each individual simulation process
    global_timeout = 150, #timeout for the entire simulation
    interpolation_timestep = 10e-12, #timestep for interpolation of data and waveforms
    mode = 'grid', #mode of simulation, can be 'grid' or 'basinhopping'
    mode_args = None, #optional: arguments for the mode, such as basinhopping arguments
    n_processes = 4, #number of processes to run in parallel
    temp_folder = 'temp_sim_files/', #folder to store temporary files such as modified circuits and output files
    waveforms = waveforms, #list of objective waveforms to match during the optimization
    randomize_params = True, #randomize the order of parameter combinations to speed up the optimization process
    ) 

It will bring up a GUI displaying the ongoing progress, including the best score (lowest/best optimizer value) and the objective waveforms for that particular parameter combination.  When finished, it will return two values representing a single dataframe with the best result, and a larger dataframe where each row is a simulation result representing a different parameter combination.

**Output data**

Spicewrapper has two significant data structures that you will receive.

1. The result dataframe (result_df)

This dataframe stores the results of individual NGSpice runs.  Each row represents the outcome of one run, and the columns contain all the important data.  The columns look like:

[index, circ_file_orig_contents, total_energy, spice_df, param1val, param2val, param3val, etc]

``index`` is the row index of the particular result. Nothing special.

``circ_file_orig_contents`` is the raw text of the circuit file used in that run.

``total_energy`` is the net energy consumed by all the DC voltage sources over the simulation window.  This calculation might fail for various reasons, most commonly when you don't have any DC voltage sources for it to calculate from.  In the future, this may be extended to other types of sources.

``spice_df`` is a dataframe itself which contains the values of all simulation variables over time.  We'll explain this later.

``paramXval`` is the value of the associated simulation parameter for this particular combination of parameter values.  If your parameter name in the netlist .cir file is actually ``rval``, then this column would be named ``rval``.  The remaining columns to the right are similar, just for the other parameter values from the simulation.

2. the spice_dataframe (spice_df)

The columns are [time, variable_name1, variable_name2, etc].  The rows are the timesteps produced by the simulation.  So you get the value of every variable at every timestep.  Note that Spicewrapper inherently interpolates timesteps along a fixed grid (that you specify in the call to ``run_spicemanager`` with the argument ``interpolation_timestep``).  

**Plotting and saving**

From here on out, you've got your data in dataframes, and you can obviously do whatever you want with it.  But we've thrown in a few convenience functions to speed some things up for beginners.  data_processing.simple_plot() and data_processing.plot_sweep_result() are discussed in ``squid_param_sweep.py`` and other examples.  




