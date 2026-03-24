# Readonly

A personal tech blog built with [Hugo](https://gohugo.io/) and the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme.

Live site: [readonly.chihyanglee.cc](https://readonly.chihyanglee.cc)

## Prerequisites

Tools are managed via [mise](https://mise.jdx.dev/). Install dependencies with:

```bash
mise install
```

This sets up Hugo (extended) and Go.

## Getting Started

Clone the repo with submodules:

```bash
git clone --recurse-submodules git@github.com:chihyanglee/readonly.git
cd readonly
```

Start the local dev server (with drafts):

```bash
hugo server -D
```

## Creating a New Post

```bash
hugo new posts/<slug>/index.md
```

This scaffolds a new post under `content/posts/<slug>/` using the archetype in `archetypes/posts.md`. The post is created as a draft by default — set `draft: false` in the front matter when ready to publish.

You can co-locate images alongside `index.md` in the post's directory.

## Building for Production

```bash
hugo
```

Output is written to `public/`.
