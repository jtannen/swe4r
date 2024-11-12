# Managing Data

My Ph.D. advisor once told me "If your analysis is complicated, you just don't have the right data." I strongly agree, and for the coding researcher I'll extend it to: "If your analytic script is complicated, you haven't sufficiently cleaned your data."

# Extract, Transform, Load
Extract, Transform, and Load (or ETL) is the industry term for taking raw data and processing it into a form useable for analysis. You begin with data in some unprocessed form, and clean it into source tables that will be more useful. Next, you transform it, creating new data concepts (aggregations, summary statistics, calculations). Finally, you load all the data (in our case, "loading" just means save it; in more advanced systems you'd push it to a cloud database) so you never have to touch the raw data again.

A few rules for your research ETL.

### Rule 1: Never, ever edit the raw data manually
When you receive raw data, save it into a directory `raw_data`, and don't modify it. I cannot stress this enough. You might be tempted to open the data in Excel and delete that blank row at the top, or fix the typo in a street name, but don't. You will regret it.

Instead, use a script to load that data, make the edits, and save it to a new file. It might feel silly to use code to fix a single typo, but when you get sent V2 of that file and have to replicate your changes--which absolutely will happen--you'll be thankful you did.

As you advance, your cleaning will go from specific to general. Instead of replacing the name of the street in row 22 (with code, of course!), you'll replace all instances of that misspelling. You'll build up a list of rules that will serve you over future datasets.

### Rule 2: Separate your cleaning from your analysis
Invest heavily in creating a clean dataset that you can depend on later. Pick nits over weird edge cases--values that are null, columns where the same thing was coded in different ways. Fix them. Do this cleaning in a separate script from your analysis, which saves the outputs that your analytic script later loads. Your analysis should only do the simplest of data processing--filtering, grouping, maybe creating some new columns--but the real data creation should go in your ETL. When you discover a data issue in your analysis, don't fix it there. Instead, *go back* to the ETL to fix it and rerun everything.

### Rule 3: Add checks to your code
Now that you're cleaning your data in code, add tests throughout it whenever a step seems thorny. Are you worried a column will have NAs that shouldn't? Throw in `assert not any(np.isna(x))`. Did you filter your table to a subset? Then `assert not df.empty()`. It's costless to add these in, and when they catch something you'll be thankful. A good rule of thumb is that whenever something goes wrong--and you've spent valuable minutes of your life debugging it--throw in a check that will catch the issue next time.

## Exercise:
Reorganize your project so the folder structure looks something like:

- raw_data/
  - ...
- processed_data/
  - ...
- scripts/
  - data_cleaning.py
  - analysis.py
- outputs/
  - ...

(If your project is sufficiently complicated, you may want a separate data_cleaning script for each step.)
