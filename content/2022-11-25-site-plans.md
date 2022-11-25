+++
title = "Web hosting is a racket"
date = 2022-11-25

[taxonomies]
tags = ["hosting", "zola", "nfsn"]

+++

# Thoughts on hosting

It turns out that in my rush to get a blog up, I failed to properly survey the webhosting landscape. After a deep dive, here is what I've learned:

# Static sites are free

If all you want to do if serve some HTML files to the rest of the world, there seem to limitless providers foaming at the mouth to do it for free. GitHub Pages, Cloudflare Pages, etc. I reckon I might go with Cloudflare pages because Cloudflare [seems to support privacy](https://1.1.1.1/) and is apparently very fast. I'm currently paying DreamHost for a lot more functionality than I'm actually using, so making the switch here is no brainer.

# So who needs paid hosting?

Anybody who is doing more than serving static files. So, for example, a website with a backend, a web application, or someone who needs more fine-grained control over the system they are running. But for the purposes of hosting a static blog, I don't need to be paying DreamHost a monthly hosting fee. And if I was going to go with a webhost in the future, it would be

# [NearlyFreeSpeech.NET](https://www.nearlyfreespeech.net/) (NFSN)

The self-described "masters of [only pay for what you use](https://www.nearlyfreespeech.net/services/pricing) hosting", NFSN is likely one of the most economical hosts around. Plus, you actually have control over the root directory, [something DreamHost doesn't allow](https://willm.how/blog/site-notes). NFSN would be great for some of the following projects I'm interested in:

- Quote blog, someting aking to [this](https://qblog.aaronsw.com/). My Google Keep note just isn't cutting anymore.
- Hosting [some sort of comment system.](https://lisakov.com/projects/open-source-comments/)
- [Hosting a Gitea](https://www.r-bloggers.com/2019/12/git-hosting-for-the-distraught-and-the-restless/)?

I don't NEED any of these things, but hosting at NFSN would provide an opportunity to practice server management. Even though they don't seem to support Docker, I could try [aviary.sh](https://github.com/frameable/aviary.sh) or even better, my friend Noah's [FetchApply](https://github.com/P5vc/FetchApply)* (that is, if he ever pushes the 2.0 update to his as-yet-to-be-released Gitea).

*This link to FetchyApply is current as of now, but it is not the destination for future updates

# Future plans

As best as I can tell, DreamHost provides DNS management for any domains I purchase through them. So the tentative plan going forward is as follows:

1. Starting hosting my static blog at blog.willm.how via Cloudflare Pages (free!)
   1. The added benefit to this is that my static site generator appears to have [good support](https://www.getzola.org/documentation/deployment/cloudflare-pages/) for Cloudflare Pages
1. Spin up a NFSN server for the hosting a comment system and anything else that comes up
1. Continue using DreamHost as my domain registrar and complimentary DNS provider

Cheers!