---
title: "A simple HTTPS reverse proxy"
subtitle: "For quick and dirty HTTPS deployments"
date: 2018-08-19T20:10:32+08:00
tags: ["HTTPS", "Go"]
---

Over the past decade, there has been a concerted push towards HTTPS across the internet. On many platforms, setting up HTTPS is usually a single click away or even automatic, in no small part thanks to the rise of Let's Encrypt making certificates much more accessible. However, in non-managed environments or in the absence of application support, it usually entails setting up a reverse proxy such as Apache or NGINX to handle HTTPS connections.

Recently I had an application which didn't support HTTPS and a valid certificate and private key, and wanted to find the most direct path to serving the application over HTTPS. This probably called for some sort of reverse proxy, but while Apache, NGINX and other solutions such as OpenBSD relayd are powerful and widely-used, configuring them is often similarly involved. Was there an easier way?

## Introducing `secure`
I ended up writing a single-minded CLI tool which takes a certificate and private key, then accepts HTTPS connections and proxies them to a upstream URL:

{{< linkpreview title="yi-jiayu/secure" description="Single-minded HTTPS reverse proxy" url="https://github.com/yi-jiayu/secure" >}} 

`secure` is mostly just a thin wrapper over what's already provided by [Go](https://golang.org/)'s excellent standard library, specifically [`httputil.ReverseProxy`](https://golang.org/pkg/net/http/httputil/#ReverseProxy) and [`http.Server`](https://golang.org/pkg/net/http/#Server.ListenAndServeTLS).

### Demo
In fact, the application I was referring to earlier was [godoc](https://godoc.org/golang.org/x/tools/cmd/godoc), Go's documentation tool. I wanted to share the HTML documentation created by godoc for some code I wrote at work, but while godoc is able to run as a HTTP server for serving documentation, it does not support HTTPS on its own. Instead, `secure` can run as a reverse proxy in front of the godoc http server and terminate TLS connections for it.

Run a godoc web server on port 6060 in the background:
```console
$ godoc -http :6060 &
```

Give the `secure` binary permission to bind to port 443 so that we don't have to run it as root:
```console
$ sudo setcap CAP_NET_BIND_SERVICE=+ep $(which secure)
```

Run `secure` with your cert file, key file and the URL of the godoc server (by default, `secure` listens for incoming HTTPS connections on all interfaces and port 443):
```console
$ secure -cert cert.pem -key key.pem http://localhost:6060
2018/08/19 16:28:18 cert-file=cert.pem key-file=key.pem listen-addr=:443 upstream-url=http://localhost:6060
```
{{% caption  %}}
I'm eagerly awaiting console lexer support in Chroma (https://github.com/alecthomas/chroma/issues/137)
{{% /caption %}}

Now visit https://localhost (or your machine's address on the network) to view godoc over HTTPS!

## Security
Of course, it's not always a good idea to reinvent the wheel, especially when there are more established and battle-tested alternatives in the field. `secure` is only meant to be an easily-deployed, temporary stand-in for a more robust reverse proxy setup (for something more ambitious, take a look at [Caddy](https://caddyserver.com/), which is also built on Go).

At the same time, it may be reassuring to know that Go's network stack is quite capable on its own:

{{< linkpreview title="Achieving a Perfect SSL Labs Score with Go"
    description="blog.bracebin.com - Adventures in software development and getting things done."
    url="https://blog.bracebin.com/achieving-perfect-ssl-labs-score-with-go" >}}
    
Using `http.ListenAndServeTLS` gets an A rating from [SSL Labs](https://www.ssllabs.com/ssltest/) out of the box, and with a bit of additional configuration it's possible to get a perfect score.

{{< linkpreview title="So you want to expose Go on the Internet"
    description="Back when crypto/tls was slow and net/http young, the general wisdom was to always put Go servers behind a reverse proxy like NGINX. That’s not necessary anymore!"
    url="https://blog.gopheracademy.com/advent-2016/exposing-go-on-the-internet/" >}}
    
Go's default TLS settings resemble the Intermediate recommended configuration of the [Mozilla guidelines](https://wiki.mozilla.org/Security/Server_Side_TLS), and can be further configured for greater security. However, a lack of timeouts makes the default configuration vulnerable to certain types of denial of service attacks .

(Unfortunately, `secure` is currently using a mostly default configuration. Setting some reasonable defaults soon should be a priority.) 

Nevertheless, these articles were written back in 2016, almost 2 years ago! Surely the story has only gotten better in the meantime 🙂 (although there is still an [open issue](https://github.com/golang/go/issues/24138) on GitHub about improving the default HTTP timeouts).

## Related
While working on `secure`, I noticed that [@inconshreveable](https://inconshreveable.com/), the creator of [`ngrok`](https://ngrok.com/), also made something similar: 
{{< linkpreview title="inconshreveable/slt"
    description="A TLS reverse proxy with SNI multiplexing in Go"
    url="https://github.com/inconshreveable/slt" >}}
    
In fact, `ngrok` itself is a useful tool that can open HTTPS tunnels to your local machine over the internet, which is often what you actually want to do when securing local services with HTTPS and a use case that `secure` only addresses a part of. I use `ngrok` myself for things like sharing a work-in-progress version of my blog.
