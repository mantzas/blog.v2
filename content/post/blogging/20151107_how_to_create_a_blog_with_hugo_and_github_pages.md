---
title: Setup a blog with Hugo and Github Pages
date: 2015-11-07T13:42:43+02:00
tags: 
  - hugo
  - blog
  - githubpages
series: Blogging series
categories:
  - blog
  - hugo
  - githubpages
---

It was long my desire to write a blog with stuff that interests me.  
Lately i was studying [Golang](https://golang.org/) and i came across [Hugo](https://gohugo.io/) which is a really nice and fast site generation utility.  
This was a great opportunity to start my own blog by using [Hugo](https://gohugo.io/) and [Github Pages](https://pages.github.com/) in order to host it. Why?

* it's free
* it's [Github](https://github.com/)
* it's easy and fast

This is a walk through on how you can have a blog easy, fast and free! Let's start! The only thing you need is:

* a Github Account
* a Domain (optional)

### Steps

The following steps are needed for the initial setup and creation of the blog:

1. [Github](https://github.com/) repository for source code of the blog
2. [Github Pages](https://pages.github.com/) repository for the generated site
3. Setup [Hugo](https://gohugo.io/)
4. Create blog
5. Publish blog to [Github Pages](https://pages.github.com/)
6. Generate content and publish
7. (Optional) Setup sub domain to point to blog

### 1. [Github](https://github.com/) repository for source code of the blog

Create a repository (public or private).

### 2. [Github Pages](https://pages.github.com/) repository for the generated site

Follow the instruction on [Github Pages](https://pages.github.com/) to create a repository with your Github username. Clone it to your local drive.

### 3. Setup [Hugo](https://gohugo.io/)

Download [Hugo](https://gohugo.io/) to your local drive. Unpack it to a folder and set the path in your OS to the executable. Almost all OS are supported!!!

### 4. Create blog

* Create a folder for your blog source code and `cd` into it.
* Execute `hugo new site .`
* Execute `git init`
* Add as remote repository the repository created in Step 1. (`git remote add origin https://github.com/{username}/{repository}.git`)
* Add `.gitignore` file to exclude the path `public/`, which is the default directory of the generated static files
* Execute `git add .`
* Execute `git commit -m "initial commit"`
* Execute `git push -u origin master`

Please refer to [Hugo's](https://gohugo.io/) documentation for generating content, using themes etc.

### 5. Publish blog to [Github Pages](https://pages.github.com/)

When we are ready to deploy our blog we do the following:

* Execute `hugo -d {path}`, where path is the cloned repository path from step 2
* `cd` into the above path
* Execute `git add .`
* Execute `git commit -m "initial commit"`
* Execute `git push origin master`

After this we can enjoy our newly created blog under `http://{username}.github.io` where username should be replaced with your [Github's](https://github.com/) username.

### 6. Generate content and publish

After our initial commit we can now generate more content and publish it (Step 5).

### 7. (Optional) Setup sub domain to point to blog

Let's assume you have a sub domain `blog.domain.com`. The only thing you need to do is to setup a CNAME entry in your DNS configuration and point it to `{username}.github.io`, add a file with name `CNAME` and content `blog.domain.com` to the root folder of the repository created in step 2, commit and you're done.

### Conclusion

With these six (the seventh is optional) easy steps we have created a fully functional and fast blog. The cherry on top: **for free!!!**  
(If we used public repositories in [Github](https://github.com/)). And if we assume that almost everybody nowadays has a domain it is free even with the 7th step!

Happy blogging!
