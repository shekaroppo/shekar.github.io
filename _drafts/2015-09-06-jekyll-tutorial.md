---
title: Jekyll Tutorial
---
Resources
[Jekyll](http://jekyllrb.com/docs/)
[Treehouse](https://teamtreehouse.com/library/build-a-blog-with-jekyll-and-github-pages/building-and-customizing-the-blog/)

### Default Variable

Insted of declaring common variable in each post or page we can declare them as default variable in _config.yml

##### Example
How to add post and page default variable in _config.yml
> Remove **author**,**layout**,**category** from post files

>Remove **layout** from pages(ex: about.md)

```
defaults:
  -
    scope:
      path: "" # an empty string here means all files in the project
      type: "posts" # previously `post` in Jekyll 2.2.
    values:
      author: Shekar
      layout: "post"
      category:  Android
  -
    scope:
      path: "" # an empty string here means all files in the project
      type: "pages" # previously `post` in Jekyll 2.2.
    values:
      layout: "page"
```    
---

### Creating Pages

1. Pages can be of type .md or .html
2. Page can be created anywhere outside _post folder
3. Permalinks applied to post will not work for pages, so we need to define permalink in pages.
```
---
title: Resources
permalink: /resources/
---
```
4. To view pages on main screen go to navigation.html and add
```
<a href="{{ site.baseurl }}/page.md/">page_name</a>
```

[Resource](http://jekyllrb.com/docs/pages/)


---

### Working with Draftt

Drafts are used for writing unfinished post.

1. Create _Draft in your project
2. Create regular post
  * Date for post name is optional
  * To preview draft
  ```
  jekyll serve --drafts
  ```
  * Can create default setting for drafts
  ```
defaults:
  -
    scope:
      path: "" # an empty string here means all files in the project
      type: "drafts"
    values:
      author: Shekar
      layout: "post"
      category:  Android
```     
---

### Adding Pagination

To add pagination we need to add configuration setting in _config file

##### Example
```
pagination: 3

```
To render pagination we need to include jekyll paginator variable

#####Example
```
Replace {% for post in site.post %} {% endfor  %}
with {% for post in paginator.post %}{% endfor  %}
```

To get Next and Older posts, copy paste pagination snippit from [jekyll doc](http://jekyllrb.com/docs/pagination/)

---

### Comment Threads
Jekyll doesn't have a commenting feature built-in, but there are a few simple solutions. In this video, you'll learn how to implement the commonly used Disqus comment system, which is free to use!

Follow these four steps

1. Create Disqus account and from settings add disqus to your site
2. Register the blog by craeting site profile
3. Genrate Disqus script based on our platform
4. Include Disqus script in our post templete disqus.html
5. Create custom default variable called comments: true in _config
6. Commonts on all post will be enable by default,to disable comment override this variable in front matter
7. Copy/pste {% include disqus.html disqus_identifier=page.disqus_identifier %} in post templete below footer

---
