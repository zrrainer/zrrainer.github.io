---
layout: post
title: web scraper shinanigans
date: 2024-04-123-05-12 15:53:00-0400
giscus_comments: true
---

As part of a larger [word2vec](https://github.com/zrrainer/word2vec) project, I am aiming to collect some data regarding the usage of the word "mulch". Inspired by a conversation with my partner about internet memes, this project aims to investigate:

1. is it possible to pinpoint the semantic meaning of a made up slang word ("mulch", for example) by feeding sample usage of said word into a word2vec model and looking at its embedding?

2. in the case of homonyms (mulch being both a. gardening material and b. internet slang), is it possible to determine which definition is intended?



I've been at this web scraper for a while, using the python library Scrapy. The scraper is simple: look through all texts within a html response. If it contains the goal word ("mulch"), the entire text chunk is stored into a local mySQL database with columns:

 id int NOT NULL auto_increment,
                            url TEXT,
                            text TEXT,
                            keywords SET("mulch"),
                            PRIMARY KEY (id)

{% raw %}
```r
  ##    url                text               keywords            
  ##   (TEXT)              (TEXT)              (SET)               ()  
  ##  1 THC                 0.82               0.893               0.890  

  #contemplating adding a date dolumn
```
{% endraw %}





launch the crawler with command "scrapy crawl [crawler name]" in the crawler directory.

2024.2.26 crawler testing phase 1

does duplicate entry into scraped_link need additional handling?
scraping all body text with response.xpath("//body//text()") is a bit ick. sometimes it returns scripts that are way too long.
error closing spider sometimes
why isnt my sql code being logged


2024.3.3

updated xpath selection. should be good now (sweating)
something about the middleware passes additional argument into close_spider() and start_request() occationally. added argument collector...i really dont want to dig into the middleware
visited_links yet to be implemented
***doesnt seem to be behaving as intended. test sites:

youtube
reddit
4chan
X (no permission)
tiktok (no permission)
iFunny (no permission)




































This post shows how to add custom styles for blockquotes. Based on [jekyll-gitbook](https://github.com/sighingnow/jekyll-gitbook) implementation.

We decided to support the same custom blockquotes as in [jekyll-gitbook](https://sighingnow.github.io/jekyll-gitbook/jekyll/2022-06-30-tips_warnings_dangers.html), which are also found in a lot of other sites' styles. The styles definitions can be found on the [_base.scss](https://github.com/alshedivat/al-folio/blob/master/_sass/_base.scss) file, more specifically:

```scss
/* Tips, warnings, and dangers */
.post .post-content blockquote {
    &.block-tip {
    border-color: var(--global-tip-block);
    background-color: var(--global-tip-block-bg);

    p {
      color: var(--global-tip-block-text);
    }

    h1, h2, h3, h4, h5, h6 {
      color: var(--global-tip-block-title);
    }
  }

  &.block-warning {
    border-color: var(--global-warning-block);
    background-color: var(--global-warning-block-bg);

    p {
      color: var(--global-warning-block-text);
    }

    h1, h2, h3, h4, h5, h6 {
      color: var(--global-warning-block-title);
    }
  }

  &.block-danger {
    border-color: var(--global-danger-block);
    background-color: var(--global-danger-block-bg);

    p {
      color: var(--global-danger-block-text);
    }

    h1, h2, h3, h4, h5, h6 {
      color: var(--global-danger-block-title);
    }
  }
}
```

A regular blockquote can be used as following:

```markdown
> This is a regular blockquote
> and it can be used as usual
```

> This is a regular blockquote
> and it can be used as usual

These custom styles can be used by adding the specific class to the blockquote, as follows:

```markdown
> ##### TIP
>
> A tip can be used when you want to give advice
> related to a certain content.
{: .block-tip }
```

> ##### TIP
>
> A tip can be used when you want to give advice
> related to a certain content.
{: .block-tip }

```markdown
> ##### WARNING
>
> This is a warning, and thus should
> be used when you want to warn the user
{: .block-warning }
```

> ##### WARNING
>
> This is a warning, and thus should
> be used when you want to warn the user
{: .block-warning }

```markdown
> ##### DANGER
>
> This is a danger zone, and thus should
> be used carefully
{: .block-danger }
```

> ##### DANGER
>
> This is a danger zone, and thus should
> be used carefully
{: .block-danger }
