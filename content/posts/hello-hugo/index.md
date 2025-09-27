---
title: "Hello Hugo"
description: "I got COVID and decided to migrate my blog to Hugo."
date: 2023-08-28T17:57:18+01:00
tags: ["hugo"]
categories: ["meta", "bop"]
series: ["Hello Hugo"]
resources:
- src: images/hoot.png
  name: hoot
---

[Eight years ago, I enumerated the reasons I shelved my Jekyll powered Github Pages site in favour of begrudgingly adopting Wordpress again](https://samnicholls.net/2015/10/19/welcome-back-wordpress/).
I had gone around the simplicity-complexity cycle once more and was yearning for more features (having previously wanted fewer features) and more flexibility (having previously desired less flexibility).

Back in the days when my job was merely to turn up to class on time and print tat in three dimensions, I had the time and energy to learn how to install and manage arbitrary tools and services.
I relished needing to install a bunch of dependencies and work out how to configure them properly.
Now that I'm a grown up and made the mistake of becoming a bioinformatician, there is more than enough dependency management in my day to day, and the last thing I need at the end of the day is more of it.

Until recently, I was still running my own email through Postfix and Dovecot; only finally relenting because I did not want to bother finding out what DKIM is[^fm].
I've long since given up on maintaining my Wordpress blog, and remembering it exists makes me anxious.

But now Twitter is collapsing in on itself like a dying star, the internet seems to have been reminded of the good old days where people ran websites all of their own.
Maybe it would be fun to publish my very important thoughts on my own corner of the internet, again.

I asked Twitter (and Mastodon) for recommendations on blog platforms du jour.
"Is Jekyll and Github Pages still fashionable?", I asked, hoping that the answer was No, because I didn't want to install the Ruby Version Manager ever again.
I made it clear that I wanted a solution with minimal features (having previously wanted more features).
[Hugo](https://gohugo.io/), was offered up as a suggestion[^bh], but I was overwhelmed with analysis paralysis and it was easier to maintain the status quo of pretending I did not have a blog.


At the tail of last week, I tested positive for COVID for the second time.
My judgement is impaired and I have forgotten that it has been two years since I last attempted to update my blog, and some five years since I wrote something I would describe as a proper post.

In the full knowledge that I am likely to go all in on this endeavour only to write one post about said endeavour and not blogging for another five years, what better time than hauled up in bed to get the blog back on the web?

---

I have taken the suggestion of Hugo seriously, not because the suggestion came from someone sensible, but because they mentioned that generally the Hugo themes supported footnotes quite nicely[^th].
With my interest piqued I scrolled through Hugo's opening gambit: it's meant to be fast. We like fast.
Maybe it's the disorientation but the Hugo website and docs felt faster than most web sites I have used recently.

The [Quick Start](https://gohugo.io/getting-started/quick-start/) was compelling enough to take it for a spin.
The `hugo` CLI installed with no drama, both on Samantha's[^sam] WSL and my Mac. I liked that.
The default config format is TOML, which is nice because I deal with the inadequacy of YAML every day.

I will likely want to write about some bad code I have found, or written.
So before getting carried away on this endeavour I needed to check the support for code snippets, which was one of the reasons I moved from a static site generator to Wordpress previously.
Hugo's [syntax highlighting](https://gohugo.io/content-management/syntax-highlighting/) nicely renders fenced code blocks using [Chroma](https://github.com/alecthomas/chroma).

{{< highlight python "linenos=inline,hl_lines=1,linenostart=19" >}}
if __name__ == "__main__":
    print("Hoot hoot, here is one now.")
{{< / highlight >}}

I previously needed to install a Wordpress plugin to accomplish this, and the editor for making those code blocks was finicky.
So far so good.

Installing a theme merely requires adding a git submodule.
Swapping between themes was a trivial configuration change and I spent more time than I care to admit switching between similar minimal monochromatic themes.
I suspect that the deeper one goes with customising themes, the more painful a switch in future may be, but anecdotally there appeared to be some common configuration conventions that carried across themes.

I settled on the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme as it offers a modern and sleek reading experience, but the header looked a little plain.
Thanks to Hugo's [template lookup order](https://gohugo.io/templates/lookup-order/), it was enjoyably straightforward to override the theme to implement a nav bar inspired by [hello-friend-ng](https://github.com/rhazdon/hugo-theme-hello-friend-ng/) to add a little flair.
This is a completely different experience to attempting to modify the theme of my Wordpress blog[^wpt].

I have semiconsciously quick started my way to a fine looking blog.
Content with the theme, now I just need some content, and to work out what to do with the posts on the existing blog.


[^fm]: If you are also still running your own e-mail, you should [click my referral link and see if FastMail](https://ref.fm/u29193504) works out for you.
[^bh]: [Thanks Bert](https://twitter.com/bert_hu_bert/status/1687932683387596800), I reserve the right to blame all this on you if I don't like it.
[^sam]: My partner's brain has also been afflicted by COVID and we are transitioning to become blog people together.
[^th]: Anyone who has taken the trouble of reading my [thesis](https://research.aber.ac.uk/en/studentTheses/computational-recovery-of-enzyme-haplotypes-from-a-metagenome) will know be familiar with my fondness for footnotes.
[^wpt]: A process so painful, that it was a contributing factor in abandoning the site. Although, this may well be an unfair comparison as I'm sure Wordpress has seen some improvements of its own in the past five years.
