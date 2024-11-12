# Modularity

The most impactful way to improve your code is by making it **modular**.

Code is modular when it's broken into self-contained units each with a clear responsibility. These modules have specific inputs and outputs, and the outputs of one module are the inputs to another. Once you have modular code, you could imagine replacing one module with a different implementation that has the same inputs and outputs, leaving the rest of your system intact.

For example, a modular `fit_model` function might train a statistical model and return a prediction function. The input would be a dataframe of training data, and the output a callable function that predicts new values. Whether inside the function is a linear or logistic model, or whether it's trained locally or in the cloud, these are implementation details that don't affect the rest of the codebase, which are only ever allowed to depend on that one output function. As long as you return a function `predict`, we're happy.

> Terms of art: We call the set of inputs and outputs to a module, along with its promised behavior, its **contract**. Replacing one module for another with the same contract is called **substitution**. 

The magic of modular design is threefold:

1. **Flexibility in Implementation**
Once you've defined a module's contract, you are free to modify its internal code. As long as the inputs and outputs are in the same format, downstream code will continue to function. And you know it's impossible for later code to depend on anything that isn't an explicit output, freeing you to confidently change the internal code.

2. **Easier Maintenance**
Modularity clearly defines where responsibility lives. When you focus on writing modular code, you are forced to put related lines of code in proximity (inside the same function, or the at least the same file). Gone are the days where a changing how a variable is coded means you need to edit line 26 but also line 145. You've isolated each decision within a well-defined part of the codebase, saving future you from hunting down every object that needs updating.

3. **Fostering of Reuse**
Modularity encourages reuse. As you define contracts between modules, you'll spot recurring tasks and shared concepts. You'll realize that you've already solved a problem, and can reuse your other function directly, or that you can slightly generalize your prior solution to handle both cases. By creating reusable functions, you reduce potential errors and increase the returns on future improvements, such as adding tests or optimizing performance.

## Levels of Modularity
Modularity exists at multiple levels, with two important forms being file modularity and function modularity.

### Script Modularity
That one giant script running your entire project? It performs many different tasks: cleaning data, fitting models, generating diagnostics, and producing plots and tables.

Script modularity breaks this up into separate scripts—`data_cleaning.py`, `fit_model.py`, `create_figures.py`. If you're annoyed that it's now hard to run the whole darn thing, you can create a `run_all.py` script that calls each script in order.

With script modularity, the inputs are data files and the outputs are other data files. Downstream scripts load the data outputs of upstream ones. In my projects, `data_cleaning.py` imports csv files from `raw_data/` and outputs csv files to `processed_data/`. Then `fit_model.py` loads in the processed data, fits a model, and saves an R (`.RDS`) or python (`.pickle`). Finally, `create_figures.py` loads the model object, calls `predict()` on it, and saves plots as `.png`s.

Some convenstions of script modularity:

1. At the top of each script, declare all of the input files. Don't load a new csv on line 300. You can do this perhaps using constants:

```python
# Top of file
{imports}

DATA_DIR = "~/path/to/my/project/raw_data/"
raw_data = load(DATA_DIR + "raw_file.csv")
census_data = load(DATA_DIR + "census_download.csv")
```

As you progress you'll learn more sophisticated ways to handle file paths, but this is pretty darn good. This pattern makes clear to Future You everything that this file depends on, and where to make updates as input data changes.

2. Save files to different folders. Use obvious names like `tmp/`, `processed_data/`, `saved_models/`, `figures/`; you'll learn to rely on those names to quickly navigate the directory.

#### Exercise:
Pause reading and go to your gigantic script and do this! With luck, it'll just be a ton of cut-pasting, and will be extremely satisfying.

### Function Modularity

The other primary form of modularity is functions. This is so important that I'll give it a new chapter.
