---
title: Getting Started with Hugo and blogdown in GitHub
author: ~
date: '2017-07-01'
slug: first-post
categories: ["R"]
tags: ["R-studio", "Hugo", "blogdown"]
draft: yes
---

# Intro
When I wanted to start a blog in git-hub first I thought Jekyll was the only option. Although Jekyll is natively supported by git-hub however it is not the only static site that git-hub can handle.
Fortuitously I stumbled into this [article] (https://apreshill.rbind.io/post/up-and-running-with-blogdown/) from www.rweekly.org. Since I do quite a lot of my work in R-studio it looked quite promising to me and I decided to use Hugo.The article explains all the concepts and also refer to great many other sources to get you started with Hugo and R-studio but the article mainly focuses on hosting the site in Netlify. Netlify offers some features not available at github and it is easier to host Hugo on Netlify but I decided just to go with the Git-hub hosting. If you choose to host in netflify I guess you can just follow the article listed above.
I tried to follow along some articles but it didn't work for me so I decided to write my own once I managed to get the Hugo up and running in Git-hub.

# Getting your hand dirty
Git-hub natively supports Jekyll, and there is difference between how Jekyll and Hugo renders the site.You can read more about it [here] (https://bookdown.org/yihui/blogdown/github-pages.html) by author of blogdown. So this is where things get dirty. There might be other (may be more efficient) approach of doing this but I followed the default structure of a Hugo website and initialize the GIT repository under the public/ directory. This might seem confusing at first but have some patience and I will show you how I did it. 

# Seting up the github repository and R-project
I assume that you already know about GitHub. GitHub offers a free sub domain username.github.io. 

1. Create on GitHub *your-project*-Hugo repository (it will host Hugo’s content)
2. Create on GitHub *your-username*.github.io repository (it will host the public folder: the static website)
3. Create a local directory for example "blog" and in terminal git clone *your-project*-hugo-url in that directory.
4. Open r-studio and create a project in the folder blog
5. In the project install the blogdown, Hugo and create a new Hugo site inside *your-project* folder

  ```r
setwd("your-project")
devtools::install_github("rstudio/blogdown")
library(blogdown)
install_hugo()
new_site()
# if you like to install your own prefred theme then you can do
install_theme("github-link to the your theme", theme_example = TRUE, update_config = TRUE)
```
Now you should already see a site generated at in r-studio and if you check your project folder you shall see files create by Hugo to serve static website.

  
# Initializing the website in github 
6. Now open the terminal or git-bash and set working directory to *your-project* and type the following command, replace the username with your username.

```bash
  git submodule add -b master git@github.com:<username>/<username>.github.io.git public
```
{{< alert info >}}if you get error then use the URL of you *your-username*.github.io like this.
{{< /alert >}}
  
```bash
 git submodule add -b master https://github.com/<username/<username>.github.io.git public
```
7. Almost done: add a deploy.sh script to your project file. Copy the code and save it as deploy.sh script in your project file and run it on the git bash or terminal. Just run bash deploy.sh in terminal. 
 
 ```
 #!/bin/bash

echo -e "\033[0;32mDeploying updates to GitHub...\033[0m"

# Build the project.
hugo # if using a theme, replace by `hugo -t <yourtheme>`

# Go To Public folder
cd public
# Add changes to git.
git add -A

# Commit changes.
msg="rebuilding site `date`"
if [ $# -eq 1 ]
  then msg="$1"
fi
git commit -m "$msg"

# Push source and build repos.
git push origin master

# Come Back
cd ..
 ```
If you didn't encountered any error then you should be able to see the site once you type in http://yourusername.github.io/ 
okay now your are almost done. If you encountered some problem do check the official [documentation] (https://gohugo.io/tutorials/github-pages-blog/), see the part where it mentiones about **Hosting Personal/Organization Pages**.  

# Setting Rstudio to serve site locally and creating your first post
Well since the article mentioned in the beginning already talks about how to do that, please start reading from [**6.2 Update project options**] (https://apreshill.rbind.io/post/up-and-running-with-blogdown/) and then finally you will get you blog up and ready in GitHub. 

~Enjoy blogging 

