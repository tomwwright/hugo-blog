---
title: 'New blog, new me'
date: 2024-05-30T22:42:37+10:00
draft: true
---

INTRO ABOUT MAKING A BLOG AGAIN

## Past instances of writing things down

I once wrote [a 6-part series](https://bitbucket.org/twwright/libgdx-labs/src/master/) of instructional tutorials for game programming in Java using [LibGDX](https://libgdx.com/) for workshops at university. These tutorials were written as `.docx`, exported as `.pdf` and distributed by... I don't actually remember how they were distributed. But it wasn't easy, or good.

{{<figure src="./libgdx.png" title="Code snippets in Word, never again" >}}

A while ago I wrote things on [Medium](https://medium.com/@tomwwright). I don't know why, given at the time I was using a lot of [Ansible](https://www.ansible.com/). Better to forget.

Medium provided a good writing experience, and the reading experience and SEO was excellent. But in returning to writing I wanted to avoid the member-only content and need-to-be-online-to-write nature of platforms like Medium.

{{<figure src="./battery-life.png" title="Impressive, I know" >}}

In 2018, following that thought of getting away from Medium, I did start and upload a site built with [Hexo](https://github.com/hexojs/hexo) but that effort doesn't really belong in this section, because I _never wrote anything on it_.

{{<figure src="./hexo.png" title="RIP" >}}

## Building this blog for writing things down

In this iteration of writing things, I had the following requirements:

- Static site
- Markdown content
- Easy to theme and customise
- Don't be shit at SEO
- Don't be shit or a pain in the neck in other ways (seriously, I will put you in the bin)

At the end of the day, here's what I considered and tried:

### Hexo

[Hexo](https://hexo.io/) is a mature Node.js option for generating a static site from Markdown. It does what it says on the tin and doesn't get in its own way. But I've been intentionally trying to branch out of the Node.js ecosystem and I didn't remember being especially impressed with the experience of using it last time so I gave this one a miss.

### Docsify

[Docsify](https://docsify.js.org/#/) is actually as magical as it sounds. Create an `index.html`, upload it along with some `.md` files and it just works. I've used it in the past to great success creating [a mini-Wiki thing](https://tworealms.tomwwright.com/) of notes and characters for a tabletop roleplaying group.

The issue with `docsify` is that, given the content is generated dynamically, the SEO is essentially non-existent. Unideal for a blog-type scenario.

### Jekyll

[Jekyll](https://jekyllrb.com/) is a popular choice and a bit of an OG, but it's Ruby. Moving along.

### Gatsby

[Gatsby](https://www.gatsbyjs.com/) is a framework for building websites with [React](https://react.dev/). Gatsby felt a bit deep and feature-rich for what I was looking for.

### VitePress (and VuePress)

[VitePress](https://vitepress.dev/) is a Vue-based static site generator. It is aimed at creating technical documentation which means out of the box it [renders really nicely from plain Markdown](https://vitepress.dev/guide/markdown). VitePress is a successor ([sort of?](https://github.com/vuejs/vitepress/discussions/548)) to VuePress, but more streamlined and less flexible.

I have used [Vite](https://vitejs.dev/) previously as a replacement for [create-react-app](https://create-react-app.dev/) tooling and had good results so I did give this one a shot. I did bootstrap something but found it difficult to customise the theme: VitePress doesn't yet have a concept of plugins or a strong ecosystem, and I found myself needing to pick up more Vue than I was looking for.

### Hugo

I settled on [Hugo](https://gohugo.io/), a popular Go option for generating static sites. The theming and customisation was super straightforward, and I have been intending to learn more Go.

###  Building a look

When I was looking at Gatsby as an option I came across this [blog starter](https://www.gatsbyjs.com/starters/gatsbyjs/gatsby-starter-blog). Something about the real minimal, single width, basic bitch nature of it spoke to me. I liked the "about the author" header that became a footer on content pages.

As a starting point in Hugo I settled on [Hugo PaperMod](https://github.com/adityatelange/hugo-PaperMod) and applied some customisation to bring it into a single width layout and introduce the "about the author" ~ego trip~ element.
