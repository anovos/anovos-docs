# Anovos Docs

This repo is where the Anovos Documentation is maintained. The docs are built using a static site generator called  [`mkdocs`](https://github.com/mkdocs/mkdocs) using the [`mkdocs-material`]((https://squidfunk.github.io/mkdocs-material/) theme and deployed to [docs.anovos.ai](https://docs.anovos.ai).

## Contributing to the Anovos Docs

### Installing MkDocs Locaally

To get started, you'll need to install `mkdocs` by running:

`pip install mkdocs`

Next, you'll need to install the `mkdocs-material` theme:

`pip install mkdocs-material`

To build the docs locally, clone the repo and, from the anovos-docs repo folder, run:

`mkdocs serve`

### Updating the Docs

We write all the in markdown as `index.md` files stored in nested folders for page hierarchy. Open the `index.md` file in the folder for the page you wish to edit. Make your changes, save the file, and commit your changes to a branch in the repo that can be reviewed and merged. 

### Adding a New Page

To add a new page, you'll need to create a folder for the page and place it within the correct folder to maintain site/page hierarchy. Create an `index.md` file in that folder where you will put the new page content. Once it is saved, be sure to go to the `mkdocs.yml` file and add it to the `nav`.

### Adding Images

All images for the site are stored in the `docs/assets`. Upload your images in this folder, then put those images in the correct pages using Markdown:

![alt-text](path-to-the-image)

### Merging and Deploying

Once your changes have been committed and pushed to the repo, create a detailed pull request for review. A maintainer from the repo will review the changes and merge your PR. The Anovos Docs repo has a GitHub Action that automatically deploys the updates to the website.

