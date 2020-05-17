+++
title = "Setting up a website (Part 1)"
date = "2020-05-17"
tags = ["domain", "linux", "namecheap", "linode"]
author = "Nathaniel M. Capule"
description = "A tutorial on how to acquire a domain name and connect it to your server. You'll need three things: a server, domain registrar and some $15 on your card."
+++

{{< toc >}}

You'll need three things: a server, domain registrar and some \$15 on your card.

# Server

We have a lot of choices here. Here are some top ones:

- [**Google Cloud Platform**](https://console.cloud.google.com) - Free \$300
  credits!
- [**AWS EC2**](https://aws.amazon.com/free) - Free tier for first year!
- [**Microsoft Azure**](https://azure.microsoft.com/account/free) - Free \$200
  credits!
- [**DigitalOcean**](https://www.digitalocean.com)
- [**Linode**](https://www.linode.com/?r=69a3930f8182950d986ff87b1c8cb59c7b785f88)
- [**Vultr**](https://www.vultr.com) - Free \$50 credits!

> Actually, you also have an option to use your local hardware too. As long as
> you have a static IP, which usually needs to be requested to your ISP.

## Which VPS should I choose?

Personally, I'd recommend GCP, AWS or MS since they have a lot of free credits
and you might want to get practice. But if you are a developer and you want a
real devbox then DigitalOcean, Linode and Vultr would be more cost-efficient in
the long term.

In this post, I'll detail how to setup a site w/ a Linode server since that's
what I'm using for this site.

## Setup a Linode server

Create an account at Linode, preferably with this link:
[Sign-up w/ free \$15 credits](https://www.linode.com/?r=69a3930f8182950d986ff87b1c8cb59c7b785f88).

{{< figure src="linode_signup.png" position="center" style="width:400px" caption="Mean green Linode theme" >}}

You'll be prompted for an email and credit/debit card for billing. PayPal isn't
allowed as far as I know.

> You might want to setup two-factor authentication too. **Authy** is a nifty
> app for this.

Once done, go to https://cloud.linode.com and create a Linode server.

Here are the specs I inputted in my machine:

|                Key | Value                 | Notes                                                      |
| -----------------: | :-------------------- | :--------------------------------------------------------- |
| Distribution image | Debian 10             | Not really important. All choices are UNIX-based anyways.  |
|             Region | Tokyo, JP             | I'm in the Philippines so I chose SEA region               |
|        Linode Plan | Standard Linode 16 GB | It's \$80, but honestly 8GB and 4GB aren't bad choices too |
|               Tags | devbox                | Just for fun, I added tag called `devbox` to my instance   |

> For other cloud hosting providers, you'll probably be given almost the same
> sets of knobs you can configure.

Don't forget to provide decent **Root Password**.

If you are familiar with SSH keys, I highly advise that you set it up as well.
Check out this guide on how to set it up on Linode:
[Use Public Key Authentication with SSH](https://www.linode.com/docs/security/authentication/use-public-key-authentication-with-ssh/).

### SSH

Once you're done with all this setup, you're ready to SSH into your Linode
instance.

{{< figure src="linode_list.png" position="center" caption="I'm a Discworld fan" >}}

You have two options, use the Linode built-in console (easiest) or SSH on your
own command line (better).

- **Option 1**: Open https://cloud.linode.com/linodes, click your instance, then
  click on **Launch Console**.

- **Option 2**: Know the IP of your server, it's on
  https://cloud.linode.com/linodes. Open your terminal, then type the following:

  ```shell
  $ # The password is whatever you placed in the Root Password field previously.
  $ ssh root@123.1.2.3
  ```

  Of course, substitute 123.1.2.3 for the IP of your server.

### Mosh (Optional)

[Mosh](https://mosh.org/) is a better and faster SSH. If you are like me, with a
shitty internet connection (as usual in the Philippines), then I also recommend
setting up a Mosh client.

Check out this link: [Getting Started with Mosh](https://mosh.org/#getting)

# Domain Name Registrar

Here's the step where you give a name to your website. Because you don't access
Facebook by it's IP address (e.g. 69.63.176.13), but by it's URL name
(facebook.com).

Here are my top choices:

- [**GoDaddy**](https://ph.godaddy.com)
- [**Namecheap**](https://www.namecheap.com)
- [**Google Domains**](https://domains.google)

I'd prefer Google Domains but it's not available in the Philippines. So I used
Namecheap instead.

## Setup a domain with Namecheap

Sign-up to [Namecheap](https://www.namecheap.com) and go to
[Namecheap domain search](https://www.namecheap.com/domains/domain-name-search/).

> You might also want to enable 2FA for namecheap too.

The UI is very straightforward, so go ahead and pick your own domain name.

{{< figure src="namecheap_search.png" position="center" caption="nmcapule.dev is mine." >}}

## Choosing a Domain Name

I'm building a website for myself as a developer so I'd rather not use `.com` or
`.org`.

If I am slightly rich, I would have chosen `.io`, but I am not, so I used `.dev`
which is still a pretty cool TLD and it's fucking **cheap** (around \$15 for 1
year)!

So yah, I'd recommend `.dev` for now.

# Linking up your Domain to your Linode server

## Point Namecheap nameservers to Linode domain nameservers

Configure your domain in Namecheap's domain list view, usually at
https://www.namecheap.com/domains/list/. Just click the `Manage` button at the
right side.

{{< image src="namecheap_manage.png" position="center" style="width: 400px" >}}

Then in the property `NAMESERVERS`, select `Custom DNS` and fill up the list
with:

- ns1.linode.com
- ns2.linode.com
- ns3.linode.com
- ns4.linode.com
- ns5.linode.com

{{< image src="namecheap_linode.png" position="center" style="width: 520px" >}}

## Configure Linode domains to point to your Linode server

Open your Linode domains view at https://cloud.linode.com/domains and click
**Add a Domain**.

{{< image src="linode_domain.png" position="center" style="width: 400px" >}}

Here are the specs I used:

|                    Key | Value                         | Notes                                        |
| ---------------------: | :---------------------------- | :------------------------------------------- |
|                 Domain | nmcapule.dev                  | Substitute with your Namecheap domain name   |
|              SOA Email | _check pic_                   | Substitute with your personal email          |
|                   Tags | devbox                        | Optional, just to align with my instance tag |
| Insert Default Records | Pick "from one of my Linodes" |                                              |
|                 Linode | _linode name_                 | Pick the Linode server you created earlier   |

## Wait for DNS propagation

This usually takes an hour or two. Go eat lunch or something.

> Note that propagation time might be different for some continents. It's
> possible that your domain is ok on Asia, but not yet on America.

If you are impatient, go try pinging your domain name. If propagated, then it'll
show the IPv4 or IPv6 address of your Linode instance.

```shell
$ ping nmcapule.dev
PING nmcapule.dev(2400:8902::f03c:92ff:snip:snip (2400:8902::f03c:92ff:snip:snip)) 56 data bytes
64 bytes from 2400:8902::f03c:92ff:snip:snip (2400:8902::f03c:92ff:snip:snip): icmp_seq=1 ttl=64 time=0.021 ms
^C
--- nmcapule.dev ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.021/0.021/0.021/0.000 m
```

# Results

If setup correctly, you should now be able to:

- SSH directly to your instance via domain name, ala:

  ```shell
  $ ssh username@nmcapule.dev
  ```

- Access your website's HTTP server (e.g. http://nmcapule.dev) if there's an
  nginx daemon automatically running in your instance.

  If that is not installed, install it via:

  ```shell
  $ sudo apt install nginx
  $ sudo service nginx start
  ```

## Wait, I'm still encountering an SSL error when I access my website.

Do you mean this "Your connection is not private" error?

{{< figure src="not_private.png" position="center" style="height: 240px"
    caption="Yeah I got this error on my first try too" >}}

Well, it turns out that most modern browsers will refuse to render non-https
websites nowadays.

You have two options:

1. **The short-term solution**: If you are using _Google Chrome_, type this
   while the focus is on the browser: `thisisunsafe`. It'll let you bypass the
   error.

2. **The long-term solution**: Do it the proper way by setting up SSL certs for
   your website. [Let's Encrypt](https://letsencrypt.org/getting-started/) is
   the easiest way to do that. I'm going to detail how to setup a blog in my
   next posting and that includes setting up SSL certs.

# Appendix of other things to try

- Setup HTTPS for your website via Let's Encrypt.
- Setup terminal tools for your instance. If you're curious, here's my first few
  command lines when I ssh'd to my server instance.

  ```shell
  $ # Install essentialss
  $ sudo apt install byobu zsh git build-essential
  $ # Start zsh
  $ zsh
  $ # Enable byobu on startup
  $ byobu-enable
  $ # Install Oh-my-zsh
  $ sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
  $ git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
  $ # Install brew
  $ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
  $ echo 'eval $(/home/linuxbrew/.linuxbrew/bin/brew shellenv)' >> /home/nmcapule/.zprofile
  $ # Install Golang
  $ brew install go
  $ go --version
  ```
