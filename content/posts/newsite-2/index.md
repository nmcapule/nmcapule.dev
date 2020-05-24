+++
title = "Setting up a website (Part 2)"
date = "2020-05-23"
tags = ["domain", "linux", "hugo", "letsencrypt", "ssl"]
author = "nats"
description = "A tutorial on how to setup a static blog with Let's Encrypt and Hugo"
+++

{{< toc >}}

Yes you have setup a HTTP server on your VPS from [Part 1](/posts/newsite), but
it's not web-ready yet because:

1. Your website isn't in HTTPS
2. Your website has zero content

# Setting up nginx

Now, if you have followed all the steps from the previous tutorial, you should
already have nginx setup on your server. To check, try `curl`ing your machine's
port 80:

```shell
$ curl localhost:80
```

If you setup nginx correctly then this should output an html code in the
terminal. Otherwise, you'll get an error like:

```
curl: (7) Failed to connect to localhost port 802: Connection refused
```

If you did get this error, then let's install nginx via typing the commands
below into your server's ssh session:

```shell
$ sudo apt install nginx
$ sudo service nginx start
```

If you accessed your site's via ip+port (e.g. Type `123.1.2.3:80` in your
browser, if your IP is 123.1.2.3), it would look like this:

{{< figure src="nginx.png" position="center" style="width:400px" caption="Cool default page from nginx" >}}

# Setting up HTTPS

It's 2020 so websites that purely serve via HTTP aren't hip anymore; and more
importantly, most modern browsers will warn visitors of these kinds of websites.

In the midling phases of the internet, most technologist (and specially online
fintechs) realized that HTTP is a horribly insecure protocol for transferring
data in the internet. A simple packet sniffing from a bad actor in the same
network could reveal a user's username and password and steal the user's
identity. This is a big no-no, and lo, HTTPS just presented a solution to
encrypt data before it goes through the internet to prevent simple hacking
attempts.

Just a few points you might want to remember to differentiate between these HTTP
and HTTPS:

- HTTP is unsecure. HTTPS encrypts data before passing it to the internet.
- HTTP is served from port 80. HTTPS is served from port 443.
- HTTP URLs start with `http://` (e.g. http://google.com). HTTPS URLs start with
  `https://` (e.g. https://google.com)
- Most websites nowadays redirect visitors from their HTTP endpoints into their
  more secure HTTPS endpoints. For example, typing in `http://google.com` will
  redirect you to `https://google.com`.

> If you see a website that deals with money but notice that the URL is in http,
> stay away. Stay far far away.

## Creating a certificate via Let's Encrypt!

> For this guide, I'm using Debian linux, and basically following this
> [little guide](https://certbot.eff.org/lets-encrypt/debianstretch-nginx.html)
> here from certbot's website.

Moving your site to HTTPS is easy with Let's Encrypt's certbot.

To start, type this in your server's ssh session:

```shell
$ sudo apt install certbot python-certbot-nginx
$ sudo certbot --nginx
```

The last command will edit your server's nginx default config file, located at
`/etc/nginx/sites-available/default`.

In there, you'll see something like:

```conf
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/html;

        # Add index.php to the list if you are using PHP
        index index.html index.htm index.nginx-debian.html;

        ...snip...
}
server {
        root /var/www/html;

    ...snip...

    listen [::]:443 ssl ipv6only=on; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ...snip...
}
server {
    if ($host = nmcapule.dev) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    ...snip...
}
```

Quick explanations:

- The first server block is the default HTTP server.
- The second server block is added by `certbot`, specifying the HTTPS server.
- The third server block says that any request coming from HTTP will be
  redirected to the HTTPS server.

> Yep, certbot sucks at formatting the conf file, so the tab sizes are
> inconsistent.

## You're website is HTTPS ready now!

You can now access your website via `https://` protocol. So if you're website is
in mysite.dev, try typing `https://mysite.dev` from your browser!

# Creating content

By default, nginx serves whatever is in this path in your filesystem:
`/var/www/html/index.html`. It was specified in the nginx conf file earlier if
you looked carefully.

Try editing it and then accessing your website again from your browser. Here's a
perfectly valid HTML file:

```html
<html>
  <head>
    <title>Hello hello</title>
  </head>
  <body>
    <marquee>Wow look at me</marquee>
    <!--
        Yes marquee is deprecated, but your browser sucks if it doesn't
        implement it. Period
    -->
  </body>
</html>
```

If you revisited your website, this is how it would look like:

{{< figure src="look_at_me.png" position="center" caption="Not pictured: Perfectly awesome marquee animation." >}}

## Your options

Now that you know the basics of hosting a website in 2020, you're ready to
create your own. Either you:

1. Cram all your contents in this single `index.html` file.
2. Create other html files (ie. `page-2.html`) and link your `index.html` into
   these other pages.
3. Use a CMS like Wordpress if you're so sick of editing HTML files.
4. Create your Facebook and code it in brainf\*\*k.
5. ...etc etc gmg

There's a lot of ways not listed there. What I'm going to detail is how to
create a mostly static site, perfect for blogs and personal websites.

## Setting up a static site with Hugo

{{< image src="hugo.png" position="center" style="width: 400px" >}}

[**Hugo**](gohugo.io) is a fast and simple static site generator written in Go.
It's perfect for websites that don't need to save anything from the visitor;
like your portfolio or resume, unless you're into visitor counters.

### Installation

I used Linuxbrew for installing non-apt packages. I suggest you use it too,
since it is very convenient. Type this from your terminal:

```shell
$ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

And then to install Hugo itself:

```shell
$ brew install hugo
```

And to fix your PATH files:

```shell
$ test -d ~/.linuxbrew && eval $(~/.linuxbrew/bin/brew shellenv)
$ test -d /home/linuxbrew/.linuxbrew && eval $(/home/linuxbrew/.linuxbrew/bin/brew shellenv)
$ test -r ~/.bash_profile && echo "eval \$($(brew --prefix)/bin/brew shellenv)" >>~/.bash_profile
$ echo "eval \$($(brew --prefix)/bin/brew shellenv)" >>~/.profile
```

You should be able to invoke Hugo now:

```shell
$ hugo version
```

### Basic Usage

> To be honest, I'm just paraphrasing the
> [Quick Start Guide](https://gohugo.io/getting-started/quick-start/) from Hugo
> website :)) Check it out too!

So let's try creating a website from our home folder:

```shell
$ cd ~
$ mkdir -p projects/
$ cd projects/
$ # This creates a folder under $HOME/projects/my-site
$ hugo new site my-site
$ cd my-site/
$ # Add a theme
$ git init
$ git submodule add https://github.com/budparr/gohugo-theme-ananke.git themes/ananke
$ echo 'theme = "ananke"' >> config.toml
$ # Create a post
$ hugo new posts/hello.md
```

And then edit `posts/hello` to have a content like this:

```md
---
title: "My First Post"
date: 2019-03-26T08:47:11+01:00
draft: true
---

Hello world, what's **up**?
```

Now, these are all unprocessed files. We need to convert them somehow to html
files, via:

```shell
$ hugo build
```

This command will create a folder under `$HOME/projects/my-site/public`.

If you inspected this folder, you'll find a set of files that is compatible with
nginx's `/var/www/html` format.

{{< figure src="public.png" position="center" style="width:400px" caption="Like this mine here" >}}

### Serving to `/var/www/html`

You have a quick option: copy all contents of `$HOME/proejcts/my-site/public`
into `/var/www/html`, and that would serve correctly in your website.

The better option is to create a symbolic link from
`$HOME/projects/my-site/public` into `/var/www/html`. This way, any change
written by `hugo build` will be propagated automatically to `/var/www/html`.

```shell
$ rm -rf /var/www/html
$ sudo ln -s $HOME/projects/my-site/public /var/www/html
```

If you're too lazy to invoke `hugo build` every time you make a change to your
blog, you're in luck. Hugo has a `watch` option exactly for this use case.
Invoke it from your site's project folder with:

```shell
$ hugo -w
```

Yes it runs indefinitely. If you ctrl+C here, it will stop auto-building your
website.

# Appendix

After all that work, you now have a very convenient nerdy website! Congrats!

I advise that you try creating a few posts of your own, and trying out other
Hugo themes. Or find out other ways to make your blogging experience more
convenient.

> Hint: [code-server](https://github.com/cdr/code-server) is very promising! ;)

---

If you're curious how I'm editing this website, I do it with VSCode remote
features :) It's very nifty and useful. Here's how it looks like editing this
exact paragraph.

{{< figure src="inception.png" position="center" caption="Inceptionnn" >}}
