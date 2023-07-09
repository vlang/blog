
## The Article Series

> 1. [The Complete Beginner’s Guide to CLI Apps in V](/the-complete-beginners-guide-to-cli-apps-in-v/)
> 2. [Elevate Your V Project With Unit Tests](/elevate-your-v-project-with-unit-tests/)
> 3. **Setting Up GitHub Actions for V** _(You are here)_

## What is CI/CD, DevOps, etc.?

Software Development is a very involved process, with a lot of moving parts.
This is especially true when you have several developers working in one or
more teams. The ultimate objective of these endeavours is to understand the
client's requirements, make sure everybody is on the same page, and eventually,
deliver the final product.

**Continuous Integration** for the most part is about the second item on the
list: _making sure everyone is on the same page_. Individuality is important
for problem solving: it offers new angles and allows people to shine where they
are able. But it can be a problem in cases such as formatting
preferences and build environments. Homogeneity is preferred in these
instances. Therefore, the decision makers band together to lay down some
ground rules for everyone to follow. These include things such as:

1. Expected behaviour that is standardized using _tests_.
2. _Source Code formatting_ guidelines that are strictly enforced.
3. Managing build and release _environments_ with relevant dependencies.

Once all protocols are in place, it is beneficial to automate these steps.
It saves on time and effort while allowing the team to experiment without
the fear of breaking stuff. Another benefit is that it allows for faster
iterations. If something fails, the team will be notified and the regression
can be taken care of quickly. This is **Continuous Integration** in a
nutshell.

**Continuous Delivery** and **Continuous Deployment** are concerned with
automatically building the product after every change and deploying it to
the staging or production environment, respectively. This allows the
developers to just write code and push fixes whenever they want. The
"pipeline" takes care of the rest. An example of this in action would
be this very blog: we push changes to the main branch of the
[vlang-blog](https://github.com/vlang/vlang-blog/) repository, and
[Cloudflare Pages](https://developers.cloudflare.com/pages/)
deploys the changes to the production environment,
which in this case, is the [official V blog](https://blog.vlang.io/).

You can read
[Red Hat's page on CI/CD](https://www.redhat.com/en/topics/devops/what-is-ci-cd)
for more information. Also, here's their page on the umbrella topic of
[DevOps](https://www.redhat.com/en/topics/devops).

## When should you consider _not_ setting up a CI/CD environment?

A few years ago, people would argue that setting up a CI environment was
resource intensive. You would have to set up a CI server, install the
relevant software on it, setup the notification systems, and put a lot
of time into general infrastructure management.

Now it's no longer the case. Several providers, like
[GitLab](https://docs.gitlab.com/ee/ci/index.html) and
[GitHub](https://docs.github.com/en/actions),
provide first-party CI/CD services. They offer generous free-tiers which
exceed the requirements of regular developers and small businesses.

If you put in a bit of elbow grease, you can set up your own CI/CD service.
For example, a popular FOSS (Free Open Source Software) pairing is
[Woodpecker CI](https://woodpecker-ci.org/) with
[Gitea](https://gitea.io/en-us/).

You still have to write the pipelines for all your repositories, but
getting started with CI for a project is much easier than it used to be.

## Introduction to GitHub Actions

We will stick to [GitHub Actions](https://docs.github.com/en/actions) for
this tutorial because it is the most widely used CI/CD service in the
community at present. We can follow up with instructions for other CI services
if there's demand for it.

Let us start by discussing some terminology.

A _workflow_ is a set of jobs, plus a specification on when to run them,
described in [YAML](https://yaml.org/).
We define one workflow per YAML file. All workflows are stored in the
`.github/workflows` directory in your project. They are generally triggered
on every push, but they can be triggered manually, or on every
[git tag](https://git-scm.com/book/en/v2/Git-Basics-Tagging), or periodically.

There can be several _jobs_ per _workflow_. Each job is a set of steps
executed sequentially. Every _step_ is either a shell script that will
be run, or an _action_ from the marketplace. Each job is run on the same
_runner_ (think: server instance - may be a physical machine, a cloud
instance, etc.), so the data is generally persisted across the steps.
We use this to our advantage by setting up the environment the way we prefer
so that we can run the necessary commands afterwards.

An _action_ is a custom application that runs on the GitHub Actions platform.
Both free and paid offerings are available in the
[GitHub marketplace](https://docs.github.com/en/developers/github-marketplace/github-marketplace-overview/about-github-marketplace),
where users and organisations can publish their own actions.
Actions encapsulate an often repeated task in a convenient wrapper and
help us avoid a lot of boilerplate. A good example of an action is
[Setup V](https://github.com/marketplace/actions/setup-vlang), which we will
use in this tutorial.

> YAML is a whitespace sensitive language. This means the indentations
> matter in YAML files. If a file is not formatted properly, it can be
> a huge time-sink trying to fix indentation issues, which can also
> lead to improper nesting or de-nesting of declarations. It is recommended
> to use a local YAML linter or an external syntax validation service.

Now that we know some terms, let's try our hand at setting up a CI/CD
configuration for our V project.

## General Idea

We will need an existing V project to work on. In this tutorial, we will
use the sample [`geo`](https://github.com/hungrybluedev/geo) project as an
example.

Broadly, here are the steps we need to perform:

1. Install V with [Setup V](https://github.com/marketplace/actions/setup-vlang).
2. Clone the project.
3. Install dependencies if needed.
4. Make sure our code is properly formatted.
5. Ensure that our project can be built.
6. Run the unit tests, if any.
7. Optionally upload the built artifacts to be used later.

> Artifacts are files that are produced by the build process. They can be
> executables, logs, results, trained models, or any other file that has a
> significance to the end user, and can be reused after the pipeline has
> finished.
>
> Refer to the [documentation](https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts)
> for more information.

The reference workflow files are available here:
[workflows](https://github.com/hungrybluedev/geo/tree/main/.github/workflows).

## Setting Up GitHub Actions CI for V

Run `mkdir -p .github/workflows` locally, from the root of the project. Then use
the preferred editor of choice to create the `ci.yml` file in the `workflows`
directory. Alternatively, we can add a new workflow from the _Actions_ tab
of the repository.

```text
. (root of the project)
└── .github
    └── workflows
        └── ci.yml
```

Here we'll define our workflow that will be triggered on every push. We start by
adding the following to the file:

```yaml {tabWidth=2}
name: CI

on:
  push:
    branches: [main]
```

The name of the workflow is "CI". You can use a more descriptive name here.
The `on` section defines the triggers for the workflow. Here, we
trigger the workflow on every push to the `main` branch. We can also trigger
it on every tag, or on pushes to a pull request.

Next, we define the primary `build` job in the list of jobs. We define an
array called `os` in the _build matrix_ with our desired platforms. This goes
in the `strategy` section of the job. More details about build matrices can be
found in the
[official documentation](https://docs.github.com/en/actions/using-jobs/using-a-build-matrix-for-your-jobs).
Briefly, they help us reuse the same steps for different platforms by performing
substitution of variables. For example, `${{ matrix.os }}` will be replaced by
each OS we defined in the matrix, one at a time, and the job will be run for each of them.

```yaml {tabWidth=2}
# ...

jobs:
  build:
    strategy:
      matrix:
        os: ["ubuntu-latest", "macos-latest", "windows-latest"]
    runs-on: ${{ matrix.os }}
```

Note that _macos_ and _windows_ runners have lower availability than
_ubuntu_ runners. So it might be worthwhile to quickly prototype on
`ubuntu-latest` and then add `macos-latest` and/or `windows-latest` when
needed.

We can now define the _steps_ for the `build` job:

```yaml {tabWidth=2}
# ...

jobs:
  build:
    strategy:
      matrix:
        os: ["ubuntu-latest", "macos-latest", "windows-latest"]
    runs-on: ${{ matrix.os }}

    steps:
      - name: Install V
        uses: vlang/setup-v@v1
        with:
          check-latest: true

      - name: Checkout ${{ github.event.repository.name }}
        uses: actions/checkout@v2

      - name: Check if code is formatted
        run: |
          v fmt -diff .
          v fmt -verify .

      - name: Build ${{ github.event.repository.name }}
        run: v .

      - name: Run Tests
        run: v test .
```

First, we use the `vlang/setup-@v1` GitHub Action to install V for our runner.
We set `check-latest` to `true` to get the latest updates. Leave this out if
you prefer more stability. Then we clone the current project using the
`actions/checkout@v2` action. The number following the `@` symbol is the
version. It is 1 in case of the `setup-v` action, and 2 in case of the
`checkout` action. The name before the `/` is the name of the user or
organisation that published the action in the marketplace.

If our V project has any dependencies (mentioned in the `v.mod` file), we can
install them using the `v install` command:

```yaml
# ...
jobs:
  build:
    # ...
    steps:
      # ...
      - name: Checkout ${{ github.event.repository.name }}
        uses: actions/checkout@v2

      - name: Install dependencies
        run: v install

      - name: Check if code is formatted
        # ...
```

For more details on the `v install` command, run `v help install`. Since
`geo` does not have any dependencies, we skip this step in our workflow.

We can now start checking the codebase to ensure that it passes the minimum
standards we set. V ships with its own opinionated tool for formatting V source code.
Here is a sample usage:

```bash
# Format all V files recursively in this directory
# AND rewrite them
v fmt -w .
# ^^^
# You'll most likely use this all the time from the root of
# your V projects.

# Display the formatted output of a sample V file to stdout
v fmt sample.v
```

You can run `v help fmt` to get more details on all the available options.
Two more options we have at our disposal are:

1. `v fmt -diff .` which displays the difference between the original and
   formatted source, if any. Ideally, we don't want any difference before we
   push the commits.
2. `v fmt -verify .` which reports an error if it finds an unformatted
   file in the repository.

We use `|` in the `run` step to make YAML preserve the newlines and make it
run multiple shell commands sequentially. The official specification can be
consulted here: [literal-style](https://yaml.org/spec/1.2.2/#literal-style).
We can alternatively write the `run` step as:
`v fmt -diff . && v fmt -verify .`

Note that we do not automatically format the code in the CI because it might
lead to huge `diff`s later on and a lot of potentially redundant commits. It
is also easier to have our tests be _read-only_.

The source code formatting check finishes quickly. Next, we will make sure that
we can build our project. This is as simple as running the `v .` command. This
will build the project fast. If you plan to distribute the built executable,
it is recommended to use `v -prod .` to generate an optimised executable,
which takes slightly longer. You can also choose to generate optimised `-prod`
builds, only when a tag is pushed, instead of on each commit or Pull Request.

Finally, we run the unit tests (if any) to make sure that we have not
regressed. This will be shown by failing tests. V's built-in testing framework
can be used for this. It's as simple as running the command `v test .`
which runs all the unit tests in the project. For a detailed account on
writing unit tests in V, please refer to the
[previous article](/elevate-your-v-project-with-unit-tests/).

NOTE: If there are problems with flaky unit tests, a good first action
is to set the `VJOBS` environment variable to 1:

```yaml
- name: Run Tests
  run: v test .
  env:
    VJOBS: 1
```

`VJOBS` (if set) defines the maximum number of processes that V can spawn.
Unit tests are executed in parallel to speed up testing. Setting `VJOBS` to
1 removes parallelism and makes the tests execute sequentially. This is useful
if you're spawning other processes which make use of resources on disk, for example.

## Testing the CI with sample commits

The final workflow file will be as follows:

```yaml {linenos=table}
name: CI

on:
  push:
    branches: [main]

jobs:
  build:
    strategy:
      matrix:
        os: ["ubuntu-latest", "macos-latest", "windows-latest"]
    runs-on: ${{ matrix.os }}

    steps:
      - name: Install V
        uses: vlang/setup-v@v1
        with:
          check-latest: true

      - name: Checkout ${{ github.event.repository.name }}
        uses: actions/checkout@v2

      - name: Check if code is formatted
        run: |
          v fmt -diff .
          v fmt -verify .
      - name: Build ${{ github.event.repository.name }}
        run: v .

      - name: Run Tests
        run: v test .
```

All V projects are initialised with git. Therefore, all we need to do to
trigger a CI run is to make sure that the remote is pointed to a repository
on GitHub, add the `ci.yml` file, commit the changes and push.

If you want to test a pre-existing workflow (say, after cloning a project),
you might need to enable the workflows in the _Actions_ tab of the
repository. Then push a commit to test.

For testing purposes, we can push an empty commit to trigger the CI:

```bash
git commit -m "trigger CI attempt 1" --allow-empty
git push
```

For workflows triggered by pushing tags, we can tag commits using git like
this:

```bash
git tag -a v0.x.y -m "version description"
git push <remote> --tags
```

We can keep iterating, making tweaks, committing more changes, until the
workflow passes successfully. Any changes we make to the code from now on
will always be checked by the CI. If the workflow fails, we'll get an
email notification from GitHub.

## A Sophisticated Example

For now, we have only talked about Continuous Integration. As mentioned
before, we can run a separate workflow when git tags are pushed to the
remote repository. In this repository, we can do the same things as in
the regular workflow, but include some novel things such as building
an optimised executable and uploading the executable built for all
available runners for later use. The latter would be a good example of
Continuous Delivery (CD). If we add steps for authentication and
deployment of the executable, then that would be Continuous Deployment
(also CD).

An example of such a workflow can be found in the author's
[klonol](https://github.com/hungrybluedev/klonol/tree/main/.github/workflows)
project.

## Concluding Thoughts

A CI/CD setup is a great way to automate a lot of tedious work. It allows
the developers to focus on fixing bugs, adding features, and they do not
have to worry about finding regressions, or stepping on each other's toes.

This was a small example of setting up one CI workflow for a V project.
We specifically used GitHub Actions here, but the general concepts carry
over to other DevOps platforms as well.

We hope this article will prove useful to you. If you face any difficulties,
please feel free to reach out to us on the [V Discord Server](https://discord.gg/vlang),
specifically in the `#help` channel. If you have made a project in V that
you're proud of, and want to share it with the community, please consider
writing about it in the `#community` or `#showcase-discussion` channel. We're
always thrilled to have new people joining our community!
