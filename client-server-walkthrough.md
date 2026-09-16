# Putting It All Together: A Client-Server Walkthrough

## What Is a Client-Server Relationship?

Most of what you use on the internet works as **client-server**: a **server** is a machine that runs a service and waits for requests, and a **client** is whatever connects to it and asks for something. A web browser (client) requests a page from a web server; an SSH client connects to an SSH server; an SFTP client uploads a file to an SFTP server.

This guide walks through building a simple web server from nothing, using everything covered in the other guides in this repo:
- **Installing Linux** - launching an EC2 instance instead of running a local installer
- **SSH** - connecting to the server as a client
- **Software management** - installing Apache with `dnf`
- **vim** - writing a basic HTML page
- **systemd** - starting, enabling, and checking the Apache service
- **System logs** - confirming everything worked

## Step 1: Create the Server (EC2)

As covered in the installing-Linux guide, a cloud deployment replaces the interactive installer with a few launch-time choices. To follow along you would:

1. Launch an EC2 instance using an Amazon Linux AMI
2. Choose a **security group** that allows inbound traffic on port 22 (SSH) and port 80 (HTTP) - without this, no client can reach the server at all
3. Select an existing SSH **key pair** (or create one) so you can log in without a password

At this point you have a running server with nothing on it yet - just the base operating system, reachable over the network.

## Step 2: Connect as a Client (SSH)

From your own machine, you're the **client** connecting to the EC2 **server**:

```bash
ssh -i my-key.pem ec2-user@203.0.113.10
```

This matches the SSH guide - `ec2-user` is the default account on Amazon Linux, and `my-key.pem` is the private half of the key pair chosen at launch. Once connected, everything below happens on the server itself.

## Step 3: Install Apache (Software Management)

Following the software management guide, use `dnf` rather than compiling from source or hunting for an RPM manually:

```bash
sudo dnf install httpd
```

`dnf` resolves and installs everything Apache (`httpd`) needs in one command.

## Step 4: Write a Basic HTML Page (vim)

Apache's default web root on Fedora/Amazon Linux is `/var/www/html/`. Create a simple page there using `vim`:

```bash
sudo vim /var/www/html/index.html
```

Press `i` to enter insert mode, type a simple page:

```html
<html>
  <head><title>My First Server</title></head>
  <body>
    <h1>Hello from my EC2 server!</h1>
  </body>
</html>
```

Press `Esc`, then save and quit with `:wq`.

## Step 5: Start and Enable Apache (systemd)

Installing Apache doesn't start it automatically. This is exactly what the systemd guide covers - `httpd` is just another daemon managed by `systemctl`:

```bash
# Start it now
sudo systemctl start httpd

# Make sure it also starts automatically after a reboot
sudo systemctl enable httpd

# Or do both in one command
sudo systemctl enable --now httpd
```

Check that it's actually running:

```bash
systemctl status httpd
```

You should see `Active: active (running)` in the output.

## Step 6: Confirm With Logs

Rather than just assuming it worked, confirm it using the system-logs guide:

```bash
# Watch Apache's systemd-managed logs live
journalctl -u httpd -f
```

Now, as a **client**, request the page from a browser or with `curl` (from your own machine, or from the server itself using its local address):

```bash
curl http://203.0.113.10/
```

You should see your HTML come back, and a new line should appear in the `journalctl -u httpd` output showing the request was handled. If Apache also has `rsyslog`-based logging configured, you'd see similar activity in `/var/log/httpd/access_log` and `/var/log/httpd/error_log`.

## Tying It Together

```
You (client)  --ssh-->        EC2 instance (server, reachable because of the security group)
                                    |
                          dnf install httpd   (software management)
                                    |
                    vim /var/www/html/index.html   (write the content)
                                    |
                 systemctl enable --now httpd   (systemd starts + persists the service)
                                    |
                         journalctl -u httpd -f   (confirm it's working)
                                    |
You (client)  --http-->        Apache (server, responding to web requests)
```

Every piece is a small, separate skill - connecting over SSH, installing software, editing a file, managing a service, reading logs - but together they form the same basic pattern behind almost every server you'll ever work with: install the software, configure it, start and enable it as a service, and confirm it's working with logs before trusting it.

## Quick Reference

```bash
ssh -i key.pem ec2-user@server-ip        # connect to the server as a client
sudo dnf install httpd                    # install the web server software
sudo vim /var/www/html/index.html         # write a basic web page
sudo systemctl enable --now httpd         # start the service and enable it at boot
systemctl status httpd                    # check that it's running
journalctl -u httpd -f                    # watch logs live to confirm activity
curl http://server-ip/                    # act as a client and request the page
```
