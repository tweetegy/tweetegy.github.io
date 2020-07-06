### Draft posts

Jekyll by default provides a folder in which all your posts live. Within the Jekyll project, you’ll find a folder called `_posts`. To create a new post you’ll need to create a new markdown file with a file name in this format:

`YYYY-MM-DD-title.markdown`

Then add the metadata at the top of the file:

```
---
layout: post
title: 'I love pizza!'
---
```

Above content supplied thanks to [this post](https://michaelsoolee.com/jekyll-post-page/#:~:text=Posts,posts%20for%20your%20new%20blog.&text=The%20format%20is%20four%2Ddigit,with%20the%20markdown%20file%20extension.).

Then serve the app with the --drafts switch

```bash
jekyll serve --drafts
```

Open <http://localhost:4000> in your browser, and voilà - you will see the draft post on the home page!
