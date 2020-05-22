---
title: How to use LaTeX in Jekyll
subtitle:
date: '2014-07-25'
slug: use-latex-in-jekyll
categories:
  - Front End
tags:
  - LaTeX
---

Kramdown comes with optional support for LaTeX to PNG rendering via [MathJax](http://www.mathjax.org/) within math blocks. See the Kramdown documentation on [math blocks](http://kramdown.gettalong.org/syntax.html#math-blocks) and [math support](http://kramdown.gettalong.org/converter/html.html#math-support) for more details.

>The easiest way to use MathJax is to link directly to the public installation  available through the MathJax Content Distribution Network (CDN). When you use the MathJax CDN, there is no need to install MathJax yourself, and you can begin using MathJax right away.  
>The CDN will automatically arrange for your readers to download MathJax files from a fast, nearby server. And since bug fixes and patches are deployed to the CDN as soon as they become available, your pages will always be up to date with the latest browser and devices.

----------

## First. Using MathJax in a Theme File  
Most web-based content-management systems include a theme or template layer that determines how the pages look, and that loads information common to all pages. Such theme files provide a way to include MathJax in your web templates in the absence of MathJax-specific plugins for the system you are using. To take advantage of this approach, you will need access to your theme files, which probably means you need to be an administrator for the site; if you are not, you may need to have an administrator do these steps for you. You will also have to identify the right file if the theme consists of multiple files.

To enable MathJax in your web platform, add the line:

```js
<script type="text/javascript"src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
either just before the &lt;/head> tag in your theme file, or at the end of the file if it contains no &lt;/head>.
```

[Using MathJax in popular web platforms](http://docs.mathjax.org/en/latest/platforms/index.html#using-mathjax-in-a-theme-file)

**注：此为第一步，即在 \\_layout\default.html 的 &lt;/head> 标签前或者在 &lt;/body> 块内部的底部添加上述代码来在页面中开启 MathJax 功能**

## Second. 在第一步中的代码之前加入下述代码：

```js
<script type="text/x-mathjax-config">
    MathJax.Hub.Config({
        tex2jax: {
        skipTags: ['script', 'noscript', 'style', 'textarea', 'pre']
        }
    });

    MathJax.Hub.Queue(function() {
        // Fix <code> tags after MathJax finishes running. This is a
        // hack to overcome a shortcoming of Markdown. Discussion at
        // https://github.com/mojombo/jekyll/issues/199
        var all = MathJax.Hub.getAllJax(), i;
        for(i = 0; i < all.length; i += 1) {
            all[i].SourceElement().parentNode.className += ' has-jax';
        }
    });
</script>
```

Official guide: [Using in-line configuration options](http://docs.mathjax.org/en/v1.1-latest/configuration.html#config-files)  
Or be guided from other bloggers:[LaTeX Math Magic](http://cwoebker.com/posts/latex-math-magic)

## Third. Custom the CSS  
Of course, the CSS of your formular text can be redefined:

    code.has-jax {font: inherit; font-size: 100%; background: inherit; border: inherit;}

**注：此为第三步，即在 \\css\main.css 或 \\media\css\style.css 文件末尾加上上述代码**

以上即为在 Jekyll 中使用 LaTeX 的教程。

本文借鉴以下文章:  
[LaTeX Notes & MathJax in Jekyll](http://rangerway.com/way/2013/10/05/latex-note-and-jekyll/)  
[LaTeX Math Magic](http://cwoebker.com/posts/latex-math-magic)
