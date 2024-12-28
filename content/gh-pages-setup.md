Title: GitHub Pages Setup
Date: 2024-12-28
Category: Getting Started
Tags: python

Despite the fact that I have set up several sites hosted on GitHub
Pages using Pelican, I always seem to run into problems that require
me to rediscover the solution each time. So here are setup
instructions in excruciating detail. Following the steps in order
appears to be important.

# Create and clone a repository

Create the repo in GitHub first and clone it; don't create it locally
and push. After cloning, immediately create a push a gh-pages branch:

```sh
git checkout --orphan gh-pages
git rm -rf .
```

Add an `index.html`

```sh
cat > index.html<<EOF
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello</title>
</head>
<body>
    <h1>Hello, World!</h1>
</body>
</html>
EOF
```

Add, commit, push, and checkout main:

```sh
git add index.html
git commit -m 'create clean gh-pages'
git push origin gh-pages
git checkout main
```

# Configure Pages for this repository

Visit your repo page on GitHub. First, configure Pages by visiting
`Settings --> Pages` and confirm that "Build and Deployment" looks like
this:

<img src="{attach}/gh-pages-setup/pages-settings.png"/>

Also make note of the site url - we'll need this below.

I'm pretty sure that this configuration is selected automatically when
the `gh-pages` branch is created (hence the need to do the above step
early in the process).

While we are in settings, let's head off a permissions error on the
Actions build step by going to `Settings --> General --> Workflow
Permissions` and enabling read and write permissions:

<img src="{attach}/gh-pages-setup/workflow-permissions.png"/>

# Create a minimal site

Everything should now be set up on GitHub to succeed the first time
you push. Let's create a minimal site with the necessary
configuration. We'll be using `uv`, so make sure this is installed.

Create a `requirements.txt` file and set up a development environment.

```sh
echo "pelican[markdown]" > requirements.txt
uv virtualenv
source .venv/bin/activate
uv pip install -r requirements.txt
```

Now run the quickstart to create a site, choosing mostly defaults.
Note that it allows you to specify a url prefix, but it fails to
actually add this to `pelicanconf.py`, leading to much
head-scratching. This is the url that we noted in the Pages settings
above.

```
pelican-quickstart
> Where do you want to create your new web site? [.]
> What will be the title of this web site? Pelican GitHub Pages Quickstart
> Who will be the author of this web site? Noah Hoffman
> What will be the default language of this web site? [en]
> Do you want to specify a URL prefix? e.g., https://example.com   (Y/n) Y
> What is your URL prefix? (see above example; no trailing slash) https://nhoffman.github.io/pelican-gh-pages
> Do you want to enable article pagination? (Y/n)
> How many articles per page do you want? [10]
> What is your time zone? [Europe/Rome] America/Los_Angeles
> Do you want to generate a tasks.py/Makefile to automate generation and publishing? (Y/n) n
```

Open `pelicanconf.py` in your editor and add the Pages url to
`SITE_URL` (there should not be a trailing slash).

```python
SITEURL = "https://nhoffman.github.io/pelican-gh-pages"
```

Add the `Flex` theme in the config as well:

```python
THEME = "./Flex"
```

We'll provide the `Flex` theme as a git submodule (note the detail in
the Action config below required to include the submodule when the
repo is cloned in the build environment):

```sh
git submodule add https://github.com/alexandrevicenzi/Flex.git Flex
```

Add a page stub to content/ (otherwise the first build will fail):

```sh
cat > content/first_post.md<<EOF
Title: Page Stub
Date: 2024-12-25
Category: Getting Started

This is a page stub
EOF
```

Finally, save the following in .github/workflows/pelican.yml

```yaml
name: Pelican site CI

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
      with:
        submodules: true
    - uses: nelsonjchen/gh-pages-pelican-action@0.1.10
      env:
        GITHUB_TOKEN: ${{secrets.GITHUB_TOKEN}}
```

Specifying "submodules: true" in the checkout action: this ensures
that the Flex submodule is cloned.

Ignore some files:

```sh
cat > .gitignore<<EOF
output
.venv
__pycache__
EOF
```

Add, commit, and push all of these changes, and your site should build
and deploy to GitHub Pages at the expected url.

# Hints and Troubleshooting

Preview the site locally with

```sh
pelican -r -l --relative-urls
```

For images I prefer the convention of adding a subdirectory for each
page to contain associated images. So for this page:

```html
<img src="{attach}/gh-pages-setup/build-deployment.png"/>
```

If you want to inspect the output of the build action, you can pull
the contents of the `gh-pages` branch:

```sh
git checkout gh-pages
git fetch origin gh-pages
git reset --hard origin/gh-pages
```

I expect that this will be fixed soon, but there's an
[open issue](https://github.com/getpelican/pelican/issues/3431 for now)
for a problem in which the local site preview fails to reload when
there are changes. Fix this by adding the following to
`pelicanconf.py`:

```python
# fix https://github.com/getpelican/pelican/issues/3431 for now
IGNORE_FILES = []
```
