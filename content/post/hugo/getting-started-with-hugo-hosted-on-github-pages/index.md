+++
author = "Matt Daines"
title = "Getting Started with Hugo hosted on Github Pages"
date = "2021-03-06"

description = "Getting started with a static site hosted on Github Pages using Hugo static site generator."
tags = [
    "Hugo",
    "Github Pages"
]
categories = [
    "Hugo",
    "Github Pages"
]
series = ["Hugo Blog"]
aliases = ["hugo-blog"]
+++

The site you're reading now is hosted on Github for free, powered by Github Pages, Hugo (a static site generator) and [Jimmy's Stack Theme](https://github.com/CaiJimmy/hugo-theme-stack). This post will describe how to achieve either the same or similar thing for yourself!<!--more--> I've wanted a tech blog for a while I've started a few only to abandon them and start over a few months later. I just couldn't figure out how I wanted to do it. I first heard about Static blog generators through a Pluralsight video course so I started to look up how I could use a static site generator like Jekyll or Hugo to manage my own blog. I'm not a web developer so I knew I wanted something to abstract the complexity away. That's why many blogs before had failed - high maintenance. I might have finally settled on a solution!

## Installing Hugo

Now, I'm running on Windows 10 so my experience may differ to yours if you're running on a different operating system.

1) In the next step, we use [Chocolatey](https://chocolatey.org/install) to install Hugo on your machine. Chocolatey has a PowerShell one-liner that should install Chocolatey on your machine

2) Hugo's [documentation](https://gohugo.io/getting-started/installing) gives you many methods to install Hugo on your machine. I used [Chocolatey](https://gohugo.io/getting-started/installing#chocolatey-windows) to install Hugo. I had no issues here and Hugo was setup very quickly

3) If everything is setup correctly, running `hugo version` in a console should return something like: `Hugo Static Site Generator v0.78.1-347F2DE6 windows/amd64 BuildDate: 2020-11-05T09:42:11Z`

## Github Repository Setup

Github Pages can be setup either per user, organisation or project. For this site, I'm creating a user site.

1) Create a **public** Repository named `<YourGithubUserName.github.io>`. For a user site, the repository must be public.

2) Create a branch that will be used for the published site. To keep things simple, I kept the default `gh-pages`.

3) Go to the repository settings, then find the Github Pages section and select the Source branch you just created. For the directory, I'm using `/(root)`.

4) Not required but if it's not enabled, why not enable `Enforce HTTPS` while we're here.

    Note: *I haven't yet implemented a custom DNS name for the site. That's on my to do list and will be a separate post!*

5) Clone your new repository to your machine and open a console at the root of the local repository

## New Hugo Site

1) With your console open and at the root of your new repository run `hugo new site .`

The `.` represents this directory. If you prefer, you could run `hugo new site blog` to build the static site in a new directory under the repository root. If you opt to do this, check that your file paths are updated to reflect the change.

## Building and running the site locally

It's likely that you'll want view your site as you go through and make changes. Of course, you don't need to but it's a good way of being able to tell when something has broken. All of these commands should be run on the root of the site. For me, that's at the repository root.

- `hugo` will just build the site
- `hugo server` will build changed parts of the site and serve the site on [localhost:1313](http://localhost:1313/)
- `hugo server --disableFastRender` will build the entire site and serve on [localhost:1313](http://localhost:1313/)

I typically run the third command. As my site is currently quite small, it takes less than a second for the site to rebuild.

## Gitignore

If you ran `hugo` or `hugo server` by now you might have noticed that you have a load of changes. These aren't required and are the generated when the site builds. To get around this I two directories to my .gitignore: `/public` and `/resources`. You may choose to add more but those two reduced the noise significantly.

## Adding a Hugo Theme

There are a large number of Hugo Themes available. Always check the theme's documentation to get started. Generally, from what I've seen the process is similar though. I use the [Stack Hugo Theme](https://github.com/CaiJimmy/hugo-theme-stack) and the guide will work to implement this theme.

1) Choose your [Hugo Themes](https://themes.gohugo.io)

2) Clone your chosen theme into your themes directory: `git clone https://github.com/CaiJimmy/hugo-theme-stack/ themes/hugo-theme-stack`.

3) Add a your theme as a submodule `git submodule add https://github.com/CaiJimmy/hugo-theme-stack/ themes/hugo-theme-stack`

    This step is important as it'll ensure you're cloning the latest theme whenever you publish a new post. I guess another way to do this is to this is to run step 2 whenever you want to update the theme.

4) Delete the `config.toml` where you first created the site.

5) Copy `themes/hugo-theme-stack/exampleSite/config.yaml` to the site root.

    This is where you can change various settings for your site. Feel free to take a look now, if you'd like.

## Setting up Github Actions to publish new posts

[Hugo has a very clear guide](https://gohugo.io/hosting-and-deployment/hosting-on-github#build-hugo-with-github-action) for building a static generated site on Github Pages. This is how my Github action started. I've made some changes to Hugo's documentation such as:

- Adding `paths-ignore` to not trigger when I'm updating something that isn't related to the site.
- Updated `runs-on` to `ubuntu-latest`. If I need to change this to an older version because of some breaking change, I can. Until then though, I'd rather use the latest images.

```yml
name: github pages

on:
  push:
    branches:
      - main
    paths-ignore:
      - 'README.md'
      - '.gitignore'
      - '.gitmodules'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: "latest"

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

## References

And that's it! Here are some references that helped me get setup.

[Chocolatey](https://chocolatey.org)

[Hugo](https://gohugo.io)

[Github Pages](https://docs.github.com/en/github/working-with-github-pages)

[Hugo Themes](https://themes.gohugo.io)

[Stack Hugo Theme](https://github.com/CaiJimmy/hugo-theme-stack)

[Stack Hugo Theme Documentation](https://docs.stack.jimmycai.com/writing)

[Github Github Pages Actions](https://github.com/peaceiris/actions-gh-pages)

[Hugo Hosting on Github](https://gohugo.io/hosting-and-deployment/hosting-on-github)

[Pluralsight Course: Build a Better Blog with a Static Site Generator](https://app.pluralsight.com/library/courses/static-site-generator-build-better-blog) - Subscription likely required!
