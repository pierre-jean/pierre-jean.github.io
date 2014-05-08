---
layout: post
title: "I migrate to Jekyllrb"
date: 2014-05-08 12:55:56 +0200
comments: true
categories: [Developers tools]
---

## From Wordpress to Jekyll

I started a blog a few month ago with the framework wordpress, but I was not convinced about the user interface. For instance, here are a few drawbacks that annoyed me:

 * The default templates are a little bit primitive, and you have to download and install new ones for custom layouts (recipies, etc.)
 * Creating a template is a heavy process, requiring some knowledge in PHP language: I don't know very much about PHP, and frankly, I'm not really interested to learn it for the basic purpose of blogging. I would have prefer some more modern/sexy web technology if I have to dig into it (javascript frameworks, dart, python, ruby ...).
 * No version-control by default

Wordpress is still a really nice framework, but it didn't fit me needs. I started to look for newer blogging framework, and I discover Jekyll.

## Jekyll, the revelation

[Jekyll][1] is a static blog-aware site generator. It is really simple to setup, and extremely flexible.
Here are the main reasons you should be interested in Jekyll:

 * It uses great tools, already well-known by the developers community.
 * It is a static web site, so you can drop your site on any service that give you an accessible share from internet. It does *not request* any special language support (PHP, ruby, python...).
 * You can host it for free on the very cool service [github.io][2], and even access it with your own domain name.
 
I said Jekyll uses well know tools and language loved by developers. Here are they:
 * **[markdown][2]**: If you don't know Markdown yet, well, you should! Markdown is a text syntax that can be easily turned into html, but focus on readability in plain text format. It's really intuitive, elegant and efficient. You can find the details of the syntax [here][3].
 * **[textile][4]**: If for a any reason you prefered textile, an alternative to markdown, it is also supported by default.
 * **[YAML front-matter][5]**: Setting some YAML front-matter at the beginning of your article allow you to specify a lot of variables (the type, the title, the layout, ...), that can be used to really tune the behaviour of your site.
 * **[Ruby][6]**: Even if I'm not a ruby developper, I couldn't deny that it's a exciting language. It is also pretty easy to understand if you have to modify some plugins. Indeed, Jekyll can be extented with ruby scripts, that will add more features when you generate your site.
 * **[Git][7]**: Even if It's not mandatory, the framework is really design to integrate easily with the decentralized version control Git, which is one of the more used in recent collaborative project.
 
## Ready to use: Jekyll Bootstrap, Octopress
 
By default, jekyll comes with really simple theme, and not a lot of feature. Even if setting up new features (comments, themes, templates, ...) is straightforward, I decided to save some time and clone the git repository of [Jekyll Bootstrap][8], which include these features.

I bought on the site [wrapbootstrap.com][10] a CSS bootstrap template for blog, that I adapt for jekyll. I spent a little more time than expected to get the exact result I wanted, but it worth it!
I also had to fix a minor bug on the comments plugin provider.

I discovered a little bit too late the framework [Octopress][11], which seems exactly what I wanted, and that I would recommand to anyone starting a blog with jekyll.

## To do

I still have unfinished tasks for the blog:

 * As I create my own theme for Jekyll, there is some depencies between the theme and my layout, which shouldn't exist if I would have create a clean generic template to be deployed for any blog. I don't know if Octopress is more obvious in its architecture, but I found on Jekyll Bootstrap that it can sometimes be tricky to choose what should be in the theme and what should be in the blog layout.
 
 * I used some private plugin on Jekyll (to generate a tag cloud for instance), which isn't supported on github. I still host my code on github, but I have to publish the generated site and the source code on different git branches to have the blog accessible on [http://pierre-jean.github.io]. Actually, I host it on my own server, but it could be usefull to really host it on github.
 
 * Publishing the blog require manual step (SSH connection to my server, and run the command to pull/push from git and generate the static content): all of these could be automated.
 
 As a conclusion, I'm very happy to be able to finally write my post on markdown, and simply commit them with git. If you are a developper, you really should enjoy jekyll! If you have any advices/tweak/opinions, feel free to share them with a comment!




 
 
 
 
 
