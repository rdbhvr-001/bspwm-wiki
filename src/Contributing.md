# How to Contribute to Doc741

Doc741 exists to make **bspwm understandable**, not just configurable. Contributions that improve clarity, correctness, and learning are always welcome.
You **don't need to be an expert** to contribute - even small improvements help.

## What You Can Contribute

You can help Doc741 in many ways:
* Fix typeos or unclear explanations
* Improve wording for beginners
* Add examples that explain *why*, not just *how*
* Expand existing sections with missing details
* Suggest new sections or topics
* Share debugging tips or common pitfalls
If something confused you while learning bspwm, that's probably worth documenting.

### What Doc741 Values

Before contributing, keep these principles in mind:
1) Explain concepts, not just commands
2) Prefer reasoning over copy-paste solutions
3) Assume the reader is curious, not careless
4) Keep a neutral, respectful tone
5) Make sure that your content has clarity

<div class="admonish warn">
<p>Doc741 aims to <b>reduce blind config copying</b>, not encourage it. But this documentation shall include some configuration files and references purely for learning purposes.</p>
</div>

### How to contribute
1) Fork the repository from github : [rdbhvr-001/doc741](https://github.com/rdbhvr-001/doc741.git)
2) Create a new branch for your change in that forked repository
3) Install `rust mdbook` for documentation by the following commands

```
# Make sure that you have cargo and rust installed. A simple surf in web can help you install them.
# Install mdbook via cargo

cargo install mdbook

# Make sure to add cargo to your PATH

```

4) Learn how to use [mdbook](https://rust-lang.github.io/mdBook/).
5) In the forked repository, you will find `SUMMARY.md` file and other md files inside src directory, which is basically the index and source for Doc741, follow the instructions given in step 4.
6) After editting the markdown files or adding new ones, you can use the following command to generate the book from the project root dir.
```
# To test it in your friendly local browser test with
mdbook serve

# To build the book, in the terminal typeos
mdbook build

# The book is compiled to html and is in `./book` directory.
```

7) Rename the `./book` to `./docs`
8) Open up a pull request after testing the book locally in browser

<div class="admonish tip">
<p><b>An Easier Way to Contribute :</b><br>You can click that edit source button on the navbar near print button to directly edit the source, make a pull request and make changes.</p>
</div>
