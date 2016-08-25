---
layout: post
title: GitHub Pages tut
permalink: /github-pages-tut/
author:    locnx
keywords:  github pages jekyll linux bash
menutitle: Tech
---
[version: 0.1]

# Quickstart

- End-to-end guide to create GitHub site, setup Jekyll, config DNS, CNAME: [captainwhippet blog](http://captainwhippet.com/blog/2014/05/11/blog-setup-details.html)
- start Jekyll locally with auto regeneration for update: 
	`bundle exec jekyll s --force_polling`

# GitHub official tuts

- [create site on GitHub](pages.github.com)
- [automatic page generation](https://help.github.com/articles/creating-pages-with-the-automatic-generator)
- [setting up with Jekyll](https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/)
- [using Jekyll with Pages](https://help.github.com/articles/using-jekyll-with-pages)
- [GitHub Pages by Thinkful ](https://www.thinkful.com/learn/a-guide-to-using-github-pages/)
- [Adding a Jekyll theme to your GitHub Pages site](https://help.github.com/articles/adding-a-jekyll-theme-to-your-github-pages-site/)

# Jekyll

### Theme
- selected theme: [webjeda sidebar](https://blog.webjeda.com/jekyll-themes/sidebar/)
- alternative, too complicated: [jekyllDecent from jwillmer blog](http://jwillmer.github.io/jekyllDecent/)

### General content building guide
- [Creating and Hosting a Personal Site on GitHub](http://jmcglone.com/guides/github-pages/)
- [Get Started With GitHub Pages (Plus Bonus Jekyll)](https://24ways.org/2013/get-started-with-github-pages/)


### Installation

#### Linux

- [Install Ruby 2.2.x on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-ruby-on-rails-with-rbenv-on-ubuntu-16-04)
	
	TL;DR commands:
	
```
sudo apt-get update
sudo apt-get install autoconf bison build-essential libssl-dev libyaml-dev libreadline6-dev zlib1g-dev libncurses5-dev libffi-dev libgdbm3 libgdbm-dev
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
source ~/.bashrc
#Also adding ~/.rbenv/bin/rbenv init to your ~/.bash_profile will let you load rbenv automatically

#check rbenv
type rbenv

git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
rbenv install -l
rbenv install 2.3.1
rbenv global 2.3.1

# check ruby version
ruby -v

#install gem
echo "gem: --no-document" > ~/.gemrc
gem install bundler jekyll

rbenv rehash
```

- [Install Ruby 2.2.x on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-install-ruby-on-rails-with-rbenv-on-ubuntu-14-04):
	
	TL;DR commands:
	
```
sudo apt-get update

sudo apt-get install git-core curl zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev python-software-properties libffi-dev

cd
git clone git://github.com/sstephenson/rbenv.git .rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
echo 'eval "$(rbenv init -)"' >> ~/.bash_profile

git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bash_profile
source ~/.bash_profile

rbenv install -v 2.2.3
rbenv global 2.2.3

ruby -v

echo "gem: --no-document" > ~/.gemrc

gem install bundler jekyll

rbenv rehash
```

- Notes: if rbenv not work, try `source ~/.bashrc` or `source ~/.bash_profile`
- Install JS runtime:
	- `$ gem install execjs`
	- `$ gem install therubyracer`
	- add dependency into Gemfile bundle with this line: gem "therubyracer"

#### Ubuntu bash on Windows 10
- Install Ruby & gem as in Linux
- Problems
	- [UNRESOLVED]Failed to watch when run `Jekyll serve`:
		- Issue: https://github.com/jekyll/jekyll/issues/5233
		- Workaround: start without watching `Jekyll serve --no-watch`
	- [Permission]
	
#### Windows

Use alternatively 1 of following:
- [Official installation guide](http://jekyllrb.com/docs/windows/#installation)
- [How to install Jekyll and pages-gem on Windows (x64)](http://jwillmer.de/blog/tutorial/how-to-install-jekyll-and-pages-gem-on-windows-10-x46)
- Problems: Jekyll is not supported for Windows officially. There's some unresolved bug, i.e with '/' in file path

*Notes*: I droped this option due to it's too annoying
	


