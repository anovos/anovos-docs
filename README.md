# Anovos Docs

This is the repository for [_Anovos_](https://github.com/anovos/anovos)' documentation.

The docs are built using [`mkdocs`](https://github.com/mkdocs/mkdocs) using the [`mkdocs-material`](https://squidfunk.github.io/mkdocs-material/) theme and deployed to [docs.anovos.ai](https://docs.anovos.ai).

## Contributing to the Anovos Docs

### Setting up

To get started, first clone the repository:

`git clone https://github.com/anovos/anovos-docs.git`

Then, enter the newly created `anovos-docs` folder and install `mkdocs` and the other dependencies by running:

`pip install -r requirements.txt`

To build and serve the docs locally, run:

`mkdocs serve`

### Editing the docs

Simply edit the Markdown files or add new ones. If you added a new file and would like it to show up in the navigation bar, don't forget to add it to the `nav` section in [`mkdocs.yml`](./mkdocs.yml).

Once you're done, commit your changes and push them to a new branch in the repo so that they can be reviewed and merged. 

### Adding Images

All images for the site are stored in the `docs/assets`. Upload your images in this folder, then put those images in the correct pages using Markdown:

```markdown
![alt-text](path-to-the-image)
```

### Merging and Deploying

Once your changes have been committed and pushed to the repo, create a detailed pull request for review. A maintainer from the repo will review the changes and merge your PR. The Anovos Docs repo has a GitHub Action that automatically deploys the updates to the website.


## Updating the API documentation

The [API documentation](https://docs.anovos.ai/api/) is automatically generated. There is a [GitHub Action workflow](https://github.com/anovos/anovos-docs/actions/workflows/apidocs.yml) that can be triggered to run manually. It fetches the current version of the library, generates the API docs and opens a PR that can be reviewed and merged just as any other contribution to the docs. 
