---
layout: post
title: GitHub Pages tut
permalink: /github-pages-tut/
author:    locnx
tags: github jekyll linux markdown
category: Features
---

### Quickstart

1. Create an User site on GitHub [^1]
2. Using Jekyll to generate static site hosted on GitHub Pages [^2]
3. Adding a Jekyll theme to your GitHub Pages site [^3]

DONE!

<!--more-->

### Step further

#### Basic guide

- Jekyll homepage: [https://jekyllrb.com](https://jekyllrb.com)
- Setting up site locally with Jekyll [^4]

#### Completed guides

- End-to-end guide to create GitHub site, setup Jekyll, config DNS, CNAME [^5]

>**Tips:** for proofing, start Jekyll locally with auto regeneration for update: 
	`bundle exec jekyll s --force_polling`
	
#### Jekyll Installation

##### Linux

- Install Ruby on Ubuntu 16.04 [^6]
	TL;DR commands:
	
```bash
#install rbenv
sudo apt-get update
sudo apt-get install autoconf bison build-essential libssl-dev libyaml-dev libreadline6-dev zlib1g-dev libncurses5-dev libffi-dev libgdbm3 libgdbm-dev
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
source ~/.bashrc

#install ruby
git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
rbenv install -l
rbenv install 2.3.1
rbenv global 2.3.1

#install gem
echo "gem: --no-document" > ~/.gemrc
gem install bundler jekyll

rbenv rehash
```

>**Notes**: if rbenv not work, try `source ~/.bashrc` or `source ~/.bash_profile`

- Install JS runtime:
	- `gem install execjs`{:.language-bash}
	- `gem install therubyracer`{:.language-bash}
	- add dependency into Gemfile bundle with this line: gem "therubyracer"

##### Ubuntu bash on Windows 10
- Install Ruby & gem as in Linux
- Problems
	- [UNRESOLVED]Failed to watch when run `Jekyll serve`:
		- Issue: https://github.com/jekyll/jekyll/issues/5233
		- Workaround: start without watching `Jekyll serve --no-watch`
	- Permission denied issue
	
##### Windows

Use alternatively 1 of following:
- [Official installation guide](http://jekyllrb.com/docs/windows/#installation)
- [How to install Jekyll and pages-gem on Windows (x64)](http://jwillmer.de/blog/tutorial/how-to-install-jekyll-and-pages-gem-on-windows-10-x46)
- Problems: Jekyll is not supported for Windows officially. There's some unresolved bug, i.e with '/' in file path

>**Notes**: I droped this option due to it's too annoying

#### Theme

- Reference theme: [webjeda sidebar](https://blog.webjeda.com/jekyll-themes/sidebar/),
Hyde, [jekyllDecent](http://jwillmer.github.io/jekyllDecent/)

#### Tags
- [Display all site tags](http://vrepin.org/vr/Tagging/)
- [Tags, Plugins, more features](http://charliepark.org/tags-in-jekyll/)

#### Reference building guides for Github Pages with Jekyll

- [Creating and Hosting a Personal Site on GitHub](http://jmcglone.com/guides/github-pages/)
- [Get Started With GitHub Pages (Plus Bonus Jekyll)](https://24ways.org/2013/get-started-with-github-pages/)


- - -

__Links__:

[^1]: [GitHub guide: create site on GitHub](https://pages.github.com)
[^2]: [GitHub guide: Using Jekyll to generate static site hosted on GitHub Pages](https://help.github.com/articles/using-jekyll-with-pages)
[^3]: [Adding a Jekyll theme to your GitHub Pages site](https://help.github.com/articles/adding-a-jekyll-theme-to-your-github-pages-site/)
[^4]: [GitHub guide: Setting up your GitHub Pages site locally with Jekyll](https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/)
[^5]: [End-to-end guide to create GitHub site, setup Jekyll, config DNS, CNAME:http://captainwhippet.com/blog/2014/05/11/blog-setup-details.html](http://captainwhippet.com/blog/2014/05/11/blog-setup-details.html)
[^6]: [Install Ruby on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-ruby-on-rails-with-rbenv-on-ubuntu-16-04)
