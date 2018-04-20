---
title: "My Hugo Setup"
date: 2018-04-20T13:10:41+07:00
tags: ["hugo", "deployment"]
cover: 
  image: https://raw.githubusercontent.com/gohugoio/hugoDocs/master/static/img/hugo-logo.png
---

Static blog isn't a new thing for me. I have known Jekyll, Octopress, Pelikan, Hugo and so on. Now I'm trying out Hugo, this blog you're reading is generated automatically from my [GitHub repository](https://github.com/akhy/akhy.chickenzord.com) for every pushes I make.

### the idea

As a [DevOps engineer in my daily job](https://linkedin.com/in/akhyar), I've accostumed to multi-environment setup (e.g. prod/dev environments) for deployment. Somehow I feel an urge to apply it to this blog as well \*ahemm*. The branch mapping goes like this:

- master &rarr; akhy.chickenzord.com (prod)
- develop &rarr; akhydev.chickenzord.com (dev)

The point is that `develop` is used as "draft" branch so I can try out new features/hacks that might not work on my local machine. I'd love to mindlessly push to `develop` when doing some trial-and-error hacks, sparing myself from making local machine to behaves like the live server.

Furthermore and obviously, the two environments have common configs but differs a little. For instance, the `dev` should build draft posts as well (i.e. hugo's `--buildDrafts` flag).

### implementation

For simplicity, I choose [Caddy](https://caddyserver.com/) to serve static files created by Hugo. You should give it a try too. Caddy's configuration is very simple and has built-in support for Git and webhook which are the features I appreciate the most in this setup.

```Caddyfile
{$SITE_ADDRESS} {
    root /var/www/html
    gzip

    header / {
        X-Frame-Options DENY
        Referrer-Policy "same-origin"
        X-XSS-Protection "1;mode=block"
    }

    log stdout
    errors stdout

    git {
        repo github.com/akhy/akhy.chickenzord.com
        branch {$GIT_BRANCH}
        path /var/www/app
        hook /webhook {$WEBHOOK_SECRET}
        then git submodule update --init --recursive
        then hugo -v {$HUGO_OPTS} --destination=/var/www/html
    }
}
```

Notice the variables? I make it like that so the environment can have different configs, hence the name "environment variables". For instance, GIT_BRANCH has the value `develop` in `dev`, while `master` for `prod`. You got the idea.

The interesting part is in the `git` block. Basically it sets source dir in `/var/www/app` and do Hugo build to `/var/www/html` whenever a webhook call is made to `/webhook` endpoint. [Kudos to Aleksandr Tihomirov for the idea in his blog post](https://zeta.pm/blog/building-this-blog/)!

So my blog repository on GitHub has two webhooks:

- https://akhy.chickenzord.com/webhook
- https://akhydev.chickenzord.com/webhook

Every pushes to the repo will trigger both environments to re-generate blog from their respective branches.

### deployment

I run Caddy servers (yes, not just one) with above configuration behind reverse proxy running on Docker. I won't explain it in details, but the general structure goes like this:

{{% center %}}
![Final Setup](/img/hugosetup.svg)
{{% /center %}}

That's it.

### what's next?

I'm still hacking around to add [Isso commenting system](https://posativ.org/isso/) to this blog. It's a self-hosted alternative to Disqus or IntenseDebate, so I'll write another blog post about getting it works.

Cheers!!

---

Related links

- https://zeta.pm/blog/building-this-blog/ (I take the idea from here)
- https://github.com/chickenzord/docker-caddy-hugo
- https://hub.docker.com/r/chickenzord/caddy-hugo/
