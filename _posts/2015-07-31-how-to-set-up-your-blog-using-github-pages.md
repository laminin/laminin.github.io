---
layout:     post
title:      How to setup your own blog using github pages and Jekyll
date:       2015-07-31 19:31:19
author:     Franklin
summary:    An article about how to setup your own blog and deploy it on github pages.
categories: jekyll github-pages
#thumbnail:  heart
tags:
 - Setting up
 - jekyll
 - github-pages
---

I always wanted to create my own blog and publish it on any server, but i didnt know where to start?, how to start?. Recently i heard about *Jekyll* and *GitHub-Pages* then i gave it a try, it turned out very easy, hence i am writing  **How to setup your own blog using github pages and Jekyll**  

<ins>Step 1:</ins>
Create a User or Organization site from [github-pages.](https://pages.github.com/) Do not use project site.

<ins>Step 2:</ins>
Clone and navigate to the repo by using the following commands.
{% highlight sh %}
git clone https://github.com/username/username.github.io
cd username.github.io
{% endhighlight %}

<ins>Step 3:</ins>
Install the [*Ruby*](https://www.ruby-lang.org/en/downloads/) and [*Ruby gem package manager.*](https://rubygems.org/pages/download)

<ins>Step 4:</ins>
Install jekyll by using the following command.
{% highlight sh %}
sudo gem install jekyll
{% endhighlight %}

<ins>Step 5:</ins>
Create a new blog by following command.
{% highlight sh %}
jekyll new .
{% endhighlight %}
Start the local server by following command.
{% highlight sh %}
jekyll serve
{% endhighlight %}
Now browse to http://localhost:4000

<ins>Step 6:</ins>
If you don't like the default them, you can download other themes from [here.](http://jekyllthemes.org/)
Replace all the content with downloaded content.

<ins>Step 7:</ins>
Configure your blog's title, contact details, Social network contact details in \_config.yml file
Configure Header, footer in *\_includes/footer.html*, *\_includes/header.html*.

<ins>Step 8:</ins>
Create a new post under \_posts folder. Use standard naming convention (YYYY-MM-DD-name-of-the-post.md).
Customize the post as you wish.

<ins>Step 9:</ins>
Add, commit, push your changes to github by the following commands.
{% highlight sh %}
git add .
git commit -m "deploying blog "
git push origin master
{% endhighlight %}

<ins>Step 10:</ins>
As soon as you push your code to github pages, you can see your blog live at http://username.github.io/
