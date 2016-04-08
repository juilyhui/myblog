```toml
# post title, required
title = "Welcome"

# post slug, use to build permalink and url, required
slug = "welcome"

# post description, show in header meta
desc = "welcome to izgnod's blog"

# post created time, support
# 2015-11-28, 2015-11-28 12:28, 2015-11-28 12:28:38
date = "2016-04-08 00:00:00"

# post updated time, optional
# if null, use created time
# update_date = "2016-03-25 12:20:20"

# author identifier, reference to meta [[author]], required
author = "izgnod"

# thumbnails to the post
thumb = "@media/golang.png"

# tags, optional
tags = ["welcome"]
```
#### Content

The content is data after first block. All words will be parsed as markdown content.

```markdown

When you read the post, `PuGo` is running successfully.

This post is generated from file `source/welcome.md`. You can learn it and try to write your own article with following guide.

...... (markdown content)

Markdown is a lightweight markup language with plain text formatting syntax designed
so that it can be converted to HTML and many other formats using a tool by the same name.
Markdown is often used to format readme files, for writing messages in online discussion forums,
and to create rich text using a plain text editor.

```
