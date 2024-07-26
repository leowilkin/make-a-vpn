# Build your own VPN

## Description
Learn how to setup a remote server, on a cloud hosted platform, and install a custom Wireguard software using Cloudflare DNS and device filtering

## Pre-requisites
- A stable WiFi connection
- A computer with a terminal
- About an hour and a half

## Definitions
- VPN: Virtual Private Network, think NordVPN, or ExpressVPN. A platform which routes your internet traffic through a dedicated server to increase privacy and prevent snooping by your ISP or School. (also handy for getting around github.com blocks!)
- DNS: Domain Name Server, the platform that gets the IP address from the website you enter for your computer to fetch data from. For example, the website coolcats.com might have the IP address 133.209.181.146. Your browser needs the IP address (like a post code, or a ZIP code) to get the HTML file  (what you see on a website).
- Network ISP: Internet Service Provider - if you're in the US, think Verizon & similar broadband providers; if you're in Europe, think Vodaphone, BT (UK). The company that provides you with your connection to the wider world. However, they can route your internet traffic through their own filtering, which can lead to privacy issues!
- Wireguard: An open-source VPN protocol maintained by the open-source community. We'll be using an opinionated fork, called wg-easy.

## Why?
Why host your own VPN server? Well, to begin with, it's a cool and easy project to introduce you to Docker, and cloud server platforms like DigitalOcean, AWS, or Akamai Linode. Also, you can increase your network privacy by preventing your cell provider or network ISP from snooping on your network traffic, or geoblocking content on platforms like Netflix.

Also, if you're at school/college/high school, your sysadmin might block services like GitHub, HackClub, or Repl.it. This isn't very fair, but you can circumvent these blocks (in most circumstances) with a VPN. Even better, if you connect multiple devices to it (like multiple members of a Hack Club), you can share your open ports when you run webapps like React, Astro, or vanilla HTML. BONUS!

But wait - there are already solutions like NordVPN, ExpressVPN, or free ones like xVPN, so why build your own? Well - free VPNs are privacy sinkholes and most likely collect and sell all your data to the highest bidder.

> Remember, when the product is free, you're the product!

Also, VPNs that you pay for are expensive and may limit how many devices you can connect. The cheapest plan I found (billed monthly) was NordVPN for $12/month! Yours will be COMPLETELY free for _2_ months, and after that will only be $5/month.

> Okay, quit dithering - let's do it!

## Steps

What will we need?

Firstly, we're going to need a server to run the code on, and to route traffic through. We're also going to need the software for the application, WireGuard. Finally, I'll show you how to use this VPN on your phone, laptop, and even some routers!

### Setup the server

We're going to be using Akamai Linode, because it's quite cheap (and they give you $100 for 60 days, no credit card required!) What Linode do is they let you use some of their infrastructure across the world for a marginal fee (my personal VPN runs for $5/month)

1. Swing on by to linode.com/$speciallink$ to get $100 free credit for 60 days - you don't have to enter any card details, and they won't charge you the second your credit runs out. Also, don't worry about running out of credit - your server only costs something like $0.0008 per hour.

2. Once you've finished the set-up process, and SET UP 2FA ON YOUR ACCOUNT, click on `Create` - it's a blue button next to the search bar - and choose Linode.

3. For the software image, choose `Debian 12`, as this is the system which runs best for the Wireguard fork we're using.

4. On the Region tab, you have two options. What you choose depends on your use case. For example, I live in London and I'm not going to be accessing geoblocked content (where a platform like Netflix only lets you access a certain show in the US), so I pick London, UK - but if I wanted a VPN in the US, I'd pick something like New York, or Dallas. The benefits of choosing a server location in another country is that you can access geoblocked content easily, however you will find that it's a bit slower, and you may experience video streaming issues (cause your server is a hella of a way away). TL;DR I'd advise picking whichever location is closest to where you'll be using it 90% of the time, but if you want to access geoblocked content like Netflix, choose something like the US, or Canada.

5. Now we select which type of server we want. You will probably not need that much CPU or storage (my server never goes above 15% CPU usage, and I've got 4-5 devices running consistently through it). Choose the `Shared CPU` tab, and the `Nanode 1GB` option. This is the cheapest plan possible on Linode, and will suit you well.

6. Give it a name in the label section (I've named mine wg-easy, which is the name of the software), and a password. This is what you'll use to connect to the server via your terminal.

BONUS: If you know what you're doing, I'd suggest adding an SSH key for the device you're using as it makes it just a tiny bit quicker to log in.

7. Ignore everything else, and hit `Create`!

### Log into your server

Now, we've gotten our server provisioned, and the little status icon should change from `Booting` to `Running`. Give it a couple of minutes, stretch your legs, have a cup of strong, English tea, and then continue this tutorial.

1. Go onto your terminal on your computer. If you're a Windows user, type <kbd>Win</kbd> + <kbd>R</kbd>, then in the dialog box, type <kbd>cmd</kbd> and click Enter! If you're on MacOS, it's <kbd>Command</kbd> + <kbd>Space</kbd>, and type `Terminal`. If you're on Linux, you should know how to open a terminal. If not, google it...

2. Go back to your Linode page, and at the top on the right you'll see a command that looks something like this: 

```bash
ssh root@133.209.181.146
```

Click on it to copy it, and paste this into your terminal.

3. If you set an SSH key, you'll be logged in immediately - if not, enter the root password you set when creating your server.

NOTE: If you've forgotten this, delete your Linode, and start again from Step 2 of the last section (sorry!)

4. Now, we're going to install Docker. Docker is a nifty little (massive) tool that allows you to spin up databases and softwares from just a small command. It's amazing, and is also really cool to use.

On your server (which we've now logged into - how cool!), you're going to type the following command as one entire thing. Just copy and paste the entire thing, and hit <kbd>Enter</kbd>

```bash
curl -sSL https://get.docker.com | sh
sudo usermod -aG docker $(whoami)
exit
```

5. If it's worked, you'll be logged out and your terminal will return to your own computer. Give it a couple of seconds, then follow steps 2 & 3 above to get back in before returning for step 6.

6. We've got Docker installed - what's next? Now we're going to run the command to get Wireguard setup and running. It's just one command - through the magic of Docker - how cool is that?!

I'd suggest opening up a text editing app like Notepad, or TextEdit to change the values that I'm about to tell you.

Open up your text editor and paste this in:

```docker
  docker run -d \
  --name=wg-easy \
  -e LANG=en \
  -e WG_HOST=<🚨YOUR_SERVER_IP> \
  -e PASSWORD=<🚨YOUR_ADMIN_PASSWORD> \
  -e PORT=51821 \
  -e WG_PORT=51820 \
  -v ~/.wg-easy:/etc/wireguard \
  -p 51820:51820/udp \
  -p 51821:51821/tcp \
  --cap-add=NET_ADMIN \
  --cap-add=SYS_MODULE \
  --sysctl="net.ipv4.conf.all.src_valid_mark=1" \
  --sysctl="net.ipv4.ip_forward=1" \
  --restart unless-stopped \
  ghcr.io/wg-easy/wg-easy
```
Let's break this command down.

Firstly, we're telling `docker` to get ready - we're specifically telling it that it's going to `run` a program in `-d` mode, which is detached.

We tell it that it's name is `wg-easy`, and that it's going to be in English. If you speak another language, you can change this to the country code of your preferred language.

We now need to change some stuff. We have to tell it what your server's address is on the internet so that when we connect our laptop/phone to it, it can point our devices in the right direction - to the right server. To get our IP address, Alt tab back to Linode, and copy the IP address value, which is on the left of the SSH Access command. Paste this in to your text file, getting rid of the entire tag (< >).

We need to set a password so it's super duper safe and secure. Choose a secure, memorable password - I reccommend a passphrase like `Mysterious*-Unicorn6-Smuggler9` (complicated, but easy to remember!)

From there, leave everything else as it is, and copy it to your clipboard.

7. Paste this config file into your server's terminal, hit enter, and let the Docker magic begin.

You'll see that it's 'pulling' some files - this is the code which dictates how the server will run, and system things, like the UI for your Web Dashboard. Now, it starts to get the container ready. It will make the database, and start the Wireguard service to route your network traffic through it.

8. Once it's done, and you get an empty command line, type `docker ps`, and you should see a line which says something like `wg-easy` and Running!

YAY! Pat yourself on the back - you've done the hard bit! Now, go and get another cup of English breakfast tea, and come back and we'll do the final bit together.

### Use it!

So - you've done the server setup, and you've installed the software - now what?

1. Let's check out the Admin Dashboard. To do that, go to the IP address that Linode gave you in your browser, and stick this bit at the end `:51821`, and hit enter. It might tell you that it's insecure, but override it (it's fine).

2. If everything went tickety-boo, and you've followed my instructions to the letter (including the two cups of tea), then you should find yourself with a password box. Pop the password you set as the Admin Password in our last step in here - you might still have the config open in your text editor.

#### Laptop

1. You're in. Go ahead and click on the `+` button, and choose a name for your connection to your Laptop. We'll put your VPN on here first. Click on the Download button, and save the .conf file to your Downloads - this is the instructions for the app to connect to your server.

4. Go to wireguard.org/download, and choose the system that you're on. Complete the setup process.

5. Open the app, and click `Add Tunnel` at the bottom of the window. Select your file that you downloaded, and click Open.

6. Now, all you need to do is hit `Connect`, and you're in! If you go back to the Wireguard page, you'll see that it shows how many bytes you've sent and received since you've been connected - how sick, right?

#### Mobile

1. Go ahead and click on the `+` button, and choose a name for your connection to your phone. Click save, and click on the image of the QR code. You'll see a large QR code appear.

2. On your phone's App Store, search for `Wireguard`. Download, and install, and press the `+` in the bottom right. Choose `Add from QR Code`, and scan the QR code on your laptop's screen.

3. Click save or whatever, and then flick the toggle. You may get asked to allow Wireguard to make a VPN connection, say Yes or Allow.

4. You're in! Double check that you're receiving data by looking at your laptop's screen of the Wireguard Admin UI. You'll see the pulsing red dot, and data being transmitted by the up/down.

## Feedback

I'd love some feedback on this jam - you can DM me any time on Slack at @lkmw. If something didn't work, then send me a DM and I will try my best to help!
