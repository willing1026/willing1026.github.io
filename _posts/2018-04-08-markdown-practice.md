---
layout: post
title: markdown 연습하기
author: "Paycis"
tags: [markdown]
---

markdown 태그 연습하기
> Example post를 따라 markdown Tag를 다양하게 적용해봤습니다.




# Headers
{% highlight markdown %}
# H1
## H2
### H3
#### h4
##### H5
###### H6
{% endhighlight %}

# H1
## H2
### H3
#### h4
##### H5
###### H6

# Text formatting

{% highlight markdown %}
- **Bold**
- _Italics_
- ~~Strickethrough~~
- <ins>Underline</ins>
- <sup>Superscript</sup>
- <sub>Subscript</sub>
- Abbreviation : <abbr title="HyperText Markup Language">HTML</abbr>
- Citation: <cite>&mdash; Chester How</cite>
{% endhighlight %}

- **Bold**
- _Italics_
- ~~Strikethrough~~
- <ins>Underline</ins>
- <sup>Superscript</sup>
- <sub>Subscript</sub>
- Abbreviation: <abbr title="HyperText Markup Language">HTML</abbr>
- Citation: <cite>&mdash; Chester How</cite>

# List
하위항목 작성은 `Tab`을 이용!!

{% highlight markdown %}
1. ordered item1
    * test1
        * test1-1
            * test1-1-1
        * test1-1-2
    * test1-2
    * test1-3
        1. test1-3-1
            1. test1-3-1-1
                1. test1-3-1-1-1
2. ordered item2
3. ordered item3

* unordered item1
* unordered item2
* unordered item3
{% endhighlight %}

1. ordered item1
    * test1
        * test1-1
            * test1-1-1
        * test1-1-2
    * test1-2
    * test1-3
        1. test1-3-1
            1. test1-3-1-1
                1. test1-3-1-1-1
2. ordered item2
3. ordered item3

* unordered item1
* unordered item2
* unordered item3

# Links
Check out tale on [Github](https://github.com/chesterhow/tale).

# Images
![Placeholder image](https://placehold.it/800x400 "Placeholder image")
![Image with caption](https://placehold.it/700x400 "Image with caption")
_This is an image with a caption_

# Code and Syntax Highlighting
Use back-ticks for `inline code`. Multi-line code snippets are supported too through Pygments.

{% highlight js  linenos %}
// Sample javascript code
var s = "JavaScript syntax highlighting";
alert(s);
{% endhighlight %}

_The end_