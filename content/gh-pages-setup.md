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



