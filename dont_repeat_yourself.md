# Don't Repeat Yourself
A core rule in programming is "Don't Repeat Yourself", cited so often that it gets its own abbreviation: [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself).

My own version of this rule is that you're allowed to copy-paste code once, but not twice.

Consider this example code that is only slightly modified from something I typed in a very real project just last week:

```python
prediction_dem = where(df.prediction_dem > 0.99, 0.99, where(df.prediction_dem < 0.01, 0.01, df.prediction_dem))
prediction_rep = where(df.prediction_rep > 0.99, 0.99, where(df.prediction_rep < 0.01, 0.01, df.prediction_rep))
prediction_lib = where(df.prediction_lib > 0.99, 0.99, where(df.prediction_dem < 0.01, 0.01, df.prediction_lib))
prediction_ind = where(df.prediction_ind > 0.99, 0.99, where(df.prediction_ind < 0.01, 0.01, df.prediction_ind))
```

I've copy-pasted the code not once, not twice, but three times! It's obvious to a reader what I'm trying to do, but there's so much cruft that (a) it's hard to tell exactly what changes from line to line, and (b) your eyes glaze over the lines, potentially missing something important.

For instance, did you notice the bug? In my incessant copy-pasting, I missed replacing a `dem` in line 3. Welcome to a few hours of confused debugging.

Compare that to this code:

```python
def clip(x):
    return where(x > 0.99, 0.99, where(x < 0.01, 0.01, x))

prediction_dem = clip(df.prediction_dem)
prediction_rep = clip(df.prediction_rep)
prediction_lib = clip(df.prediction_lib)
prediction_ind = clip(df.prediction_ind)
```

I've created a function `clip` that abstracts away the clipping logic. In reading the four clipping lines, it's much easier to see what varies. No mistakes are able to hide here.

There are two additional, more subtle benefits to this code. First, defining the function has also given me the opportunity to *name* the action I'm taking. No need for a comment! I've told you with one word exactly what I'm doing, `clip`ping the value, and I'm trusting you, an intelligent human being, to figure out the rest.

Second, by creating this `clip` function, I've created infrastructure that I can use elsewhere. Wherever I want to clip, I can reuse this code. And then once I'm using it enough, it's worth investing in making this function really performant--maybe speeding it up, maybe adding test coverage. That's probably not worth my time if I use the function in only one file, but certainly is when I'm using it in twenty.

Of course, you could keep deleting repeated code. You might have noticed how often I repeat the call `clip()` above, and suggest something like this:

```python
def clip(x):
    return where(x > 0.99, 0.99, where(x < 0.01, 0.01, x))

PARTIES = ("dem", "rep", "lib", "ind")

treatments = {
    party: clip(df[f"prediction_{party}"])
    for party in PARTIES
}
```

Now I only write the call for `clip()` once. Not a single line is repeated. I find this code less readable, and in this exact case would probably prefer the four `clip()`s. But the new code does make it easier for me to add a party ("green"?) in the future. Which of these examples you prefer will likely depend on how many parties you have (I'd definitely use the latter if I had twenty), and whether you want to reuse the list of parties elsewhere. Reasonable people will disagree here. But we can all agree the first example--which, again, I really wrote--was terrible.

## Constants
Functions can absorb repeated logic, but sometimes the repeated code is "data", or information that you've hard-coded into your file. "Information" is purposefully vague here, because there are so many things you might consider data. It might be the directory where your files live, a reference date, or a list of variables to use in the model.

Consider the following code:

```python
df["age_on_election_day"] = years(date("2024-11-05") - df["date_of_birth"])
df["formatted_name"] = format_name(df["name"])
df["voted"] = ~df["vote_method"].isna()
df["registered_before_election_day"] = date("2024-11-05") > df["registration_date"]
df["moved_before_election_day"] = date("2024-11-05") > df["moved_date"]
```

You'll notice that the date of this election is hard-coded three times. I've repeated myself! If I want to run this code next year, I need to make sure I've found and changed every instance of this date. Now, imagine I have to do this across ten files.

Here's the example code using a "constant" to store the repeated date:

```python
ELECTION_DATE = "2024-11-05"

df["age_on_election_day"] = years(date(ELECTION_DATE) - df["date_of_birth"])
df["formatted_name"] = format_name(df["name"])
df["voted"] = ~df["vote_method"].isna()
df["registered_before_election_day"] = date(ELECTION_DATE) > df["registration_date"]
df["moved_before_election_day"] = date(ELECTION_DATE) > df["moved_date"]
```

I've stored the date once and use that everywhere. If I need it in another file, I can import it. Now, updating my script next year is easy.

There are a few conventions with constants that are universal, and which you'll learn to cognitively depend on.
1. Constants are named using `ALL_CAPS_WITH_UNDERSCORES`.
2. A constant should be an extremely simple object: a string, a number, or a lists strings or numbers, etc. You don't want it to be a complicated class with baked-in behaviors. Notice above that I didn't even set `ELECTION_DATE = date("2024-11-05")`, although failing to did make me repeat some `date()` code later. The simpler a constant is, the more opportunities you'll find to reuse it.
3. Constants should live towards the tops of your files. If you find yourself wanting to put one in the middle, that's a good sign you should create a new file entirely.

There's no police force that will track you down if you violate these rules, and reasonable experts will disagree, but these patterns reinforce that constants are simple parameters that toggle the rest of your logic.
