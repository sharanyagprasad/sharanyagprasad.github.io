---
title: Make a Blog Site
description: Use Jekyll to build a statis website and dephost it on GitHub for free.
author: sharanya
date: 2024-11-04 17:45:00 +0800
categories: [Getting Started, Blog]
tags: [blogging]
image:
  path: /assets/img/make_a_blog_site/kaitlyn-baker-unsplash.jpg
  alt: Photo by Kaitlyn Baker on Unsplash

---

## Creating a Site Repository

To create your site repository, choose one of the following options based on your needs:

### Option 1: Using the Starter (Easier)

This option is ideal for a streamlined setup, isolating unnecessary files and minimizing configuration.

1. Sign in to GitHub and navigate to the [**starter**](https://github.com/cotes2020/chirpy-starter).
2. Click the <kbd>Use this template</kbd> button, then select <kbd>Create a new repository</kbd>.
3. Name the new repository as `<username>.github.io`, replacing `username` with your GitHub username in lowercase.

### Option 2: Forking the Theme

If you plan to customize features or UI design extensively, forking might be more suitable. However, note that upgrades can be more complex with this approach.

1. Sign in to GitHub.
2. [Fork the theme repository](https://github.com/cotes2020/jekyll-theme-chirpy/fork).
3. Name the new repository `<username>.github.io`, replacing `username` with your GitHub username in lowercase.

## Setting Up the Environment

With your repository ready, set up your development environment. There are two primary methods:

### Using Dev Containers (Recommended for Windows)

Dev Containers create an isolated environment using Docker, preventing conflicts and managing dependencies within the container.

**Steps**:

1. Install Docker:
   - On Windows/macOS: [Docker Desktop][docker-desktop].
   - On Linux: [Docker Engine][docker-engine].
2. Install [VS Code][vscode] and the [Dev Containers extension][dev-containers].
3. Clone your repository:
   - For Docker Desktop: Open VS Code and [clone your repository in a container volume][dc-clone-in-vol].
   - For Docker Engine: Clone locally, then [open in a container][dc-open-in-container] using VS Code.
4. Wait for the Dev Containers setup to complete.

### Setting Up Natively (Recommended for Unix-like Systems)

For Unix-like systems, native setup provides optimal performance, though Dev Containers are an alternative.

**Steps**:

1. Follow the [Jekyll installation guide](https://jekyllrb.com/docs/installation/) and install [Git](https://git-scm.com/).
2. Clone your repository to your local machine.
3. If you forked the theme, install [Node.js][nodejs] and run `bash tools/init.sh` in the root directory.
4. Run `bundle` in the repository root to install dependencies.

## Usage

### Starting the Jekyll Server

To run the site locally, use:

```terminal
$ bundle exec jekyll serve
```

> For Dev Containers, run this in the **VS Code** Terminal.
{: .prompt-info }

Your local server should be available at <http://127.0.0.1:4000>.

### Configuration

Customize settings in `_config.yml` as needed. Common options include:

- `url`
- `avatar`
- `timezone`
- `lang`

### Social Contact Options

Social contact links appear at the sidebar's bottom. Enable or disable options in `_data/contact.yml`.

### Customizing Stylesheet

To tweak the stylesheet, copy `assets/css/jekyll-theme-chirpy.scss` to your
