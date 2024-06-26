---
title: 'Creating a new Hugo blog'
date: 2024-06-02T22:42:37+10:00
tags:
  - Writing
  - Tech
---

I'm not fond of the word "blog". I don't know why but it just gives me an _ick_. But there isn't a better way to describe what you're reading right now. It's a blog. My new blog. Thanks for dropping by.

<!--more-->

I find the barrier to entry is the most important thing if I'm going to get any wriing done. Given that I work in software, you might expect that I would _enjoy_ fiddling with some homespun frankenblog that is broken more often than not. But you would be wrong -- unnecessary complexity in software, and the wasted time that goes with it, makes me want to play in traffic.

Writing is something I enjoy once I sit down, but I baulk and procrastinate whenever there are bumps or points of friction in editing, publishing, and sharing what I've written.

I'm not an experienced technical writer, but we're all starting from somewhere. I thought I could share some of my experiences which informed how I designed this new blog, which uses [Hugo](https://gohugo.io/), to try and make writing simple.

## Past instances of writing things

An early piece of technical writing I did was to write [a 6-part series](https://bitbucket.org/twwright/libgdx-labs/src/master/) of instructional tutorials for game programming in Java using [LibGDX](https://libgdx.com/) for workshops at university. Stepping tactfully over whether these tutorials were any _good_, the process of writing them was a disaster. They were painstakingly written in `.docx` documents (callouts, code snippets, and all), exported into `.pdf`, and distributed by email or shared network drive.

Terrible experience, 2/10, would only try again with the boundless energy of youth.

{{<figure src="./libgdx.png" title="Manual code snippets in Word, never again" >}}

As part of my studies I produced an [honours thesis on deep reinforcement learning](https://www.linkedin.com/in/tomwwright/overlay/1484197574467/single-media-viewer/?profileId=ACoAABJ9FNMB9NKSphcuzo057sqffa4_S7x8dwI). In the sciences [LaTeX](https://www.latex-project.org/) is a common standard for publication typesetting. I wrote my thesis using [ShareLaTeX](https://www.sharelatex.com/) (now [Overleaf](https://overleaf.com)), an online platform. This simplified getting started, sidestepping a number of wooly considerations in setting up a LaTeX desktop editor, but tied me to being online whenever I was writing. No online platform could save me, however, from the experience of _writing LaTeX_ itself (_shudder_).

{{<figure src="./latex.png" title="Yep, still strikes fear into the heart years later" >}}

Some time later, I wrote some articles (or, blogged, if we must call it that) on [Medium](https://medium.com/@tomwwright). I don't know _why_, given at the time I was learning and using a lot of [Ansible](https://www.ansible.com/). Better to forget.

Medium was a successful platform because it combined a robust and elegant Markdown-ish writing experience, a great reading experience, reader comments, and excellent SEO. At the time, it was also a primarily free platform. Unfortunately, these things often have to come to an end on the modern web and a rising tide of "members only" content and monetisation pushed me away from the platform.

{{<figure src="./battery-life.png" title="Impressive, I know" >}}

Following my experiences with Medium, I explored getting away from any particular online platform by bootstrapping a static site built with [Hexo](https://github.com/hexojs/hexo). That effort doesn't really belong in this post, however, because I _never wrote anything on it_.

{{<figure src="./hexo.png" title="RIP" >}}

## Choosing Hugo for writing things down

In coming back to writing, I wanted to continue exploring what I started with Hexo: something simple, that I could host myself, and powered by simple Markdown content. I had the following requirements:

- Static site
- Markdown content
- Just host it myself, no platforms
- Render good-looking code for technical write-ups
- Easy to theme and customise
- Don't be shit at SEO
- Don't be shit or a pain in the neck in other ways (seriously, I will put you in the bin)

Here is what I considered and tried:

### Hexo

[Hexo](https://hexo.io/) is a mature Node.js option for generating a static site from Markdown. It does what it says on the tin and doesn't get in its own way. But I've been intentionally trying to branch out of the Node.js ecosystem and I didn't remember being especially impressed with the experience of using it last time so I gave this one a miss.

### Docsify

[Docsify](https://docsify.js.org/#/) is actually as magical as it sounds. Create an `index.html`, upload it along with some `.md` files, and it just works. I've used it in the past to great success creating [a mini-Wiki thing](https://tworealms.tomwwright.com/) of notes and characters for a tabletop roleplaying group.

The issue with `docsify` is that, given the content is generated dynamically, the SEO is essentially non-existent. Less than ideal for a blog.

### Jekyll

[Jekyll](https://jekyllrb.com/) is a popular choice and a bit of an OG, but it's Ruby. Moving along.

### Gatsby

[Gatsby](https://www.gatsbyjs.com/) is a framework for building websites with [React](https://react.dev/). Gatsby felt a bit deep and feature-rich for what I was looking for.

### VitePress (and VuePress)

[VitePress](https://vitepress.dev/) is a Vue-based static site generator. It is aimed at creating technical documentation which means out of the box it [renders really nicely content from Markdown](https://vitepress.dev/guide/markdown). VitePress is a successor ([sort of?](https://github.com/vuejs/vitepress/discussions/548)) to VuePress, but more streamlined and less flexible.

I have used [Vite](https://vitejs.dev/) previously as a replacement for [create-react-app](https://create-react-app.dev/) tooling and had good results so I did give this one a shot. I did bootstrap something but found it difficult to customise the theme: VitePress doesn't yet have a concept of plugins or a strong ecosystem, and I found myself needing to pick up more Vue than I was looking for.

### Hugo

I settled on [Hugo](https://gohugo.io/), a popular Go option for generating static sites. The theming and customisation was super straightforward, and I have been intending to learn more Go. :sparkles:

## Building with Hugo

When I was looking at Gatsby as an option I came across the [blog starter](https://www.gatsbyjs.com/starters/gatsbyjs/gatsby-starter-blog). Something about the minimal, single width, basic bitch nature of it spoke to me. I liked the "about the author" header that became a footer on content pages.

{{<figure src="./gatsby.png">}}

As a starting point for a Hugo theme I settled on [Hugo PaperMod](https://github.com/adityatelange/hugo-PaperMod) and applied some customisation to bring it into a single width layout and introduce the "about the author" ~~ego trip~~ element.

### Ship... Wright?

Read it like "Ship the right way" with an unhealthy dose of name-based puns. Oh, sue me, I didn't want to agonise over it.

### Publishing with AWS Amplify

The code is stored in a public [Github repository](https://github.com/tomwwright/hugo-blog) and is deployed on push using [AWS Amplify](https://aws.amazon.com/amplify/). Domain hosting is provided by [Amazon Route 53](https://aws.amazon.com/route53/).

Configuring hosting and deployment using AWS Amplify is described in the [Hugo documentation](https://gohugo.io/hosting-and-deployment/hosting-on-aws-amplify/) but out-of-the-box it does fail to account for Hugo Modules, used to provide my Hugo theme, which [requires Go to be available](https://gohugo.io/installation/linux/#prerequisites).

Following the instructions as provided results in the error described by users in [this Github issue](https://github.com/aws-amplify/amplify-hosting/issues/884#issuecomment-1761728049) that indicates Go is unavailable:

```
Error: failed to load modules: failed to download modules: binary with name "go" not found
```

The solution was to configure the Amplify build to install Go prior to running `hugo`:

{{< github-code "https://raw.githubusercontent.com/tomwwright/hugo-blog/main/amplify.yaml" >}}

### Writing content with VS Code

For the time being I am content with the writing experience of [VS Code](https://code.visualstudio.com/) thanks to [Zen Mode](https://code.visualstudio.com/docs/getstarted/userinterface#_zen-mode).

---

_That's all for now folks, thanks for reading. In returning to writing I'm acutely aware of a looming sense that I'm going to churn out some dreadfully shit content while I knock off the rust. Here's hoping it gets better if I stick with it._