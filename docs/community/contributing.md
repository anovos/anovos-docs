# Contributing to Anovos

We'd love to have you join us in making _Anovos_ the number one choice for ML feature engineering!

## 🚀 Getting Started with Anovos Development

_Anovos_ is an open source project that brings automation to the feature engineering process.

To get Anovos up and running on your local machine,
follow the [Getting Started Guide](../getting-started.md).

The [Anovos GitHub Organization](https://github.com/anovos) contains all the repositories, sample data, and notebooks
you need to start working on improvements to the library.

## 🛠 How to Get Involved

First of all: We appreciate each and every contribution, and we'd love to get your input!

Contributions to _Anovos_ can take many different shapes and forms, for example:

- Reporting a bug
- Discussing the current state of the code
- Submitting a fix
- Proposing new features
- Testing out new features
- Contributing to the docs
- Giving talks about Anovos at meetups, conferences, and webinars

If you're interested in contributing but don't quite know where to start,
please don't hesitate to [reach out to the maintainers](./communication.md).

## ⌨ Contributing Code to Anovos

[Pull requests](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests)
are the best way to propose changes to the codebase.
We follow the [GitHub Flow](https://githubflow.github.io/) pattern, and everything happens through pull requests.

We welcome pull requests by community contributors at all times.
To make it simple, please follow these steps to contribute:

1. Fork the repo you're updating and create your branch from main.
2. If you've added code, add the corresponding tests.
3. If you've changed APIs, update the documentation.
4. Ensure that the test suite passes.
5. Issue that pull request!

ℹ Any contributions you make will be under the Apache Software License 2.0.
See the [License page](../license.md) for more information.

## 📝 Conventions for Commit Messages

Help reviewers know what you're contributing by writing good commit messages.
The first line of the commit message is the subject; this should be followed by a blank line
and then a message describing the intent and purpose of the commit.
We based these guidelines on a [post by Chris Beams](https://chris.beams.io/posts/git-commit/).

When you commit, you are accepting our DCO:

> Developer Certificate of Origin
> Version 1.1
>
> Copyright (C) 2004, 2006 The Linux Foundation and its contributors.
> 1 Letterman Drive
> Suite D4700
> San Francisco, CA, 94129
>
> Everyone is permitted to copy and distribute verbatim copies of this
> license document, but changing it is not allowed.
>
> Developer's Certificate of Origin 1.1
>
> By making a contribution to this project, I certify that:
>
> (a) The contribution was created in whole or in part by me and I have the right to submit it under the open source license indicated in the file; or
>
> (b) The contribution is based upon previous work that, to the best of my knowledge, is covered under an appropriate open source license and I have the right under that license to submit that work with modifications, whether created in whole or in part by me, under the same open source license (unless I am permitted to submit under a different license), as indicated in the file; or
>
> (c) The contribution was provided directly to me by some other person who certified (a), (b) or (c) and I have not modified it.
>
> (d) I understand and agree that this project and the contribution are public and that a record of the contribution (including all personal information I submit with it, including my sign-off) is maintained indefinitely and may be redistributed consistent with this project or the open source license(s) involved.

- When you run `git commit` make sure you sign-off the commit by typing `git commit --signoff` or `git commit -s`
- The commit subject-line should start with an uppercase letter
- The commit subject-line should not exceed 72 characters in length
- The commit subject-line should not end with punctuation (., etc)

Note: please do not use the GitHub suggestions feature, since it will not allow your commits to be signed-off.

When giving a commit body, be sure to:

- Leave a blank line after the subject-line
- Make sure all lines are wrapped to 72 characters

Here's an example that would be accepted:

```
Add luke to the contributors' _index.md file

We need to add luke to the contributors' _index.md file as a contributor.

Signed-off-by: Hans <hans@anovos.ai>
```

Some invalid examples:

```
(feat) Add page about X to documentation
```

> This example does not follow the convention by adding a custom scheme of `(feat)`

```
Update the documentation for page X so including fixing A, B, C and D and F.
```

> This example will be truncated in the GitHub UI and via `git log --oneline`

If you would like to amend your commit, follow this guide: [Git: Rewriting History](https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History)
