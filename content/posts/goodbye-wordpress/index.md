---
title: "Goodbye Wordpress: Mirroring my Wordpress blog"
description: "Mirroring my Wordpress blog with wget so I don't have to use Wordpress anymore."
date: 2023-08-31T22:53:22+01:00
tags: ["hugo"]
categories: ["meta"]
series: ["Hello Hugo"]
---

Starting this incarnation of the blog obviously begs the question of what to do with the old Wordpress one.
I am aware of
[several](https://stackoverflow.com/a/43186995)
[posts](https://askubuntu.com/questions/1233623/workaround-to-install-ubuntu-20-04-with-intel-rst-systems)
[from](https://gatk.broadinstitute.org/hc/en-us/articles/360035890791-SAM-or-BAM-or-CRAM-Mapped-sequence-data-formats)
[around](https://www.irishnews.com/magazine/2017/02/10/news/this-guy-risked-his-1-149-laptop-by-putting-it-in-a-laser-cutter-and-the-results-will-surprise-you-928793/)
[the](https://www.biostars.org/p/46327/)
[internet](https://wiki.archlinux.org/title/Dell_XPS_13_(9350))
that backlink the old blog, and I will absolutely not commit the internet crime of link rot.
I do need to maintain the integrity of the existing pages, but I don't want the trouble of migrating my old posts to be a prerequisite of getting started with Hugo, as I will just choose to do neither.

I figured the first step is to rid myself of Wordpress itself, this will at least buy me time to migrate as well as mitigate my guilt for not looking after it properly.
An easy win would be to host a static copy of the Wordpress blog contents, ensuring to maintain all the existing page URLs.
To keep the old posts reachable and indexed, I'll generate a listing to link them from the new blog.

Naturally, there are various plugins to export posts from Wordpress ([even for import to Hugo](https://gohugo.io/tools/migrations/)), but I wanted to avoid even having to login to the administration interface, which would force me to face my unfinished drafts.

### Mirror mirror

I've used `wget` in the past for scraping website content and I see no reason it won't help us out here.
`wget` is one of those tools with a man page longer than any blog post I could ever write, but luckily this is a frequent use case and the `--mirror` option is provided to set up some sensible defaults for scraping.

Several articles on mirroring recommended `--convert-links` (`-k`) which rewrites references to linked content to be relative, allowing the mirrored site to work offline, but this was not needed for my case as I'll be hosting the mirror immediately at the same remote address.
I'd additionally seen suggestions to use `--adjust-extension` (`-E`), but I found in practice this did not work for me.
In cases where CSS and JS content is loaded with URL parameters, both the filenames and the hrefs to my CSS and JS files contained the `?` escape sequence `%3F` which caused problems with loading the content[^esc].

In the end, the required command was modest:

```bash
wget --mirror --page-requisites https://samnicholls.net
```

All I had to do was update the `DocumentRoot` of my Apache configuration to point at the directory created by this command for me to refresh the page and find all the CSS and JS was broken:

> Refused to execute script from '\<URL\>' because its MIME type ('') is not executable, and strict MIME type checking is enabled

I am not the first person to be bothered by [the query strings problem](https://anarc.at/services/archive/web/#the-query-strings-problem) and a way to avoid this with wget's parameters is unclear to me.
I merely decided to fix this by dropping the query parameters from the names of all the auxiliary files:

```
for f in `find . -name '*.js?*' -o -name '*.css?*'`
do
    echo $f
    RN=$(echo $f | cut -f1 -d'?')
    mv $f $RN
done
```

And with that, `wget` has banished my Wordpress based anxiety.

The existing content on `samnicholls.net` is looking very speedy now.
Even the syntax highlighting and comment plugins still work.
If you had been fooled by all the plugins working, you'd certainly be able to tell this is no longer Wordpress from the load speed.

### Custom archive

My chosen PaperMod theme [has its own archives layout](https://github.com/adityatelange/hugo-PaperMod/blob/efe4cb45161be836d602d5cd0f857e62661dae8b/layouts/_default/archives.html) which I quite like the look of.
However, the archive layout is rendered by enumerating the Hugo site contents in a Go Template, this is not readily usable to me as I want to enumerate content outside of Hugo.

I briefly considered generating stub posts in my Hugo blog to use the archive layout as-is, but instead I decided just to re-implement the straightforward template logic in a Python Jinja template, and manipulate the mirror I just downloaded to fill in the template:

```
find 2* -name 'index.html' -not -path "*/feed/*" \
    | sort -r \
    | while read line; do TITLE=$(grep '<title>' $line | sed -nE 's,.*<title>(.*) &#8211;.*,\1,p' | recode html..utf-8); echo "$line ${TITLE}"; done \
    | python to_archive.py > ~/projects/hugo/samposium/content/v1.md
```

The only interesting thing about this rather ugly command is the use of `recode` to unescape the post titles pulled out from the `<title>` tags.
Conveniently, the rest of the post metadata (year, month and date) can be found from the directory layout[^meta].
There's not much to say about the implementation for generating the post list, but for anyone interested, `to_archive.py` and the Jinja template [has been uploaded as a gist](https://gist.github.com/SamStudio8/06ea0656d15d88367713f4307a51b019).

To render, I created a very stripped down version of the single post layout (removing breadcrumbs and other gubbins) in `layouts/_default/old_archive.html` to match the "old_archive" layout I had specified in the front matter.

One snag, to use HTML directly in a markdown file you need some means to permit the rendering of raw HTML content (which is ignored by default).
It is possible to enable this globally, but that seemed like an overreaction to just get this page working.
I found a nice solution that [suggested the use of a custom shortcode](https://discourse.gohugo.io/t/enable-unsafe-true-for-a-single-page/24415/4) on the Hugo discourse forum:

```
$ cat layouts/shortcodes/unsafe.html
{{ .Inner }}
```

I could have avoided this trouble and inserted the generated archive contents directly into the layout rather than using a content post, but we've learned something.

You can now check out [Samposium /v1](/v1) for my old posts.

Now that we have a means to reach old posts and keep them indexable by crawlers, I can now safely replace the samnicholls.net home page with this Hugo blog.
We are one giant leap closer to a freshly abandoned blog.




[^esc]: I am not entirely sure what controls this behaviour.
[^meta]: Even though I generally argue that using the file system to communicate metadata is a crime.
