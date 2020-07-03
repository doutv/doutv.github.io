---
layout: hexo
title: hexo-deployment
date: 2020-06-22 12:00:28
tags: hexo
---

# .travis.yml
官方教程有点问题，因为现在Github Pages只能部署在master分支上，所以源文件(markdown/theme)要放在其他分支(hexo)，使用Travis CL或者其他CL自动化部署时，要将目标分支`target_branch`设为`master`。

```yml
sudo: false
language: node_js
node_js:
  - 10 # use nodejs v10 LTS
cache: npm
branches:
  only:
    - hexo  # build on hexo branch
script:
  - hexo generate # generate static files
deploy:
  provider: pages
  skip-cleanup: true
  github-token: $GH_TOKEN
  keep-history: true
  on:
    all_branches: true # solve a permission problem
  target_branch: master 
  local-dir: public
```

# 文章截断
似乎没有自动截断的功能，需要手动截断，在截断的地方加`<!-- more -->`