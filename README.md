# gaoooyh.github.io
Use github pages build a blog.

view the [demo](https://gaoooyh.github.io)

***

## how to start

1. select a theme in [theme store](http://jekyllthemes.org/)
2. unzip the theme.zip and copy files to your repo.
3. fill the file `_config.yml` like

```yaml
name: Gaoooyh's 
author: gaoooyh
url: https://gaoooyh.github.io
baseurl: 
resume_site: about
description: this is a description.
github_username: gaoooyh
github: https://www.github.com/gaoooyh
plugins: [jekyll-paginate]
permalink: /:year-:month-:day-:title
paginate: 12
paginate_path: "/page/:num/"
exclude: ['README.md', 'Gemfile.lock', 'Gemfile', 'Rakefile']
highlighter: rouge
markdown: kramdown
comments :
  gitalk :
    clientID : xxx
    clientSecret : xxx
    repo : lightfish-zhang.github.io
    owner : lightfish-zhang
    admin : lightfish-zhang

```
  > how to get gitalk clientID/Secret?
  > Settings-> Developer settings->Oauth Apps

4. add your post in path `./_post`, format : 

```md
---
layout: post
title: A Example Post
date:   1970-01-01 00:00:00 +0800
category: tutorial
thumbnail: /style/image/thumbnail.jpg
icon: book
---


* content
{:toc}

## sub title

page...

## about thumbnail

add the thumbnail url

## about icon

such as book, code, web, chat, note, game, link, design, image
```


## Developer

- [chakhsu](https://github.com/chakhsu)
- [lightfish-zhang](https://github.com/lightfish-zhang)

## Thanks

- [jekyll](http://jekyllrb.com) git page engine
- [pinghsu](https://github.com/chakhsu/pinghsu), a typecho theme, it's a great design.
- [gitalk](https://github.com/gitalk/gitalk) git page comment engine, it depends on github issue.
- [smoothscroll](https://www.smoothscroll.net/mac/) SmoothScroll will give your mouse wheel (Finder, Safari, Chrome, etc.) buttery smooth scrolling


