---
layout: post
title: Accepting specific self-signed server certificate on Git
comments: true
permalink: "accepting-specific-self-signed-server-certificate-on-git"
---

Today, I've been facing the problem about how to accept a self-signed server certificate for [Git][git] repositories using **HTTPS URLs**. You can be wondering why I don't use SSH connection and the problem is resolved, isn't? Well, I can't use it this time so I had to find a way to accept it permanently.

First option is disable certificates verification when I execute a [Git][git] command:

```
GIT_SSL_NO_VERIFY=true git clone https://www.myserver.com/myrepo.git
```

That is ok and it works but you can't always add the `GIT_SSL_NO_VERIFY` on all git commands when you're using any CI system or, in my case, when using [CocoaPods][pods].

# Insecure way

Another easy way is telling to [Git][git] don't verify SSL certificates for all repositories:

```
git config --global http.sslVerify false
```

Please, **DON'T DO THAT** or you'll take away all security offered by SSL.

Now, the right approach is configure [Git][git] to accept only your self-signed certificate

# Smart way

Summary:

 1. Download the self-signed certificate
 2. Tell to [Git][git] to trust this certificate


### Downloading the self-signed certificate

If you have `openssl` installed you can do it easily:

```
openssl s_client -connect myserver.com:443
```

Catch the response and create a file only with the lines between `BEGIN CERTIFICATE` and `END CERTIFICATE` (both lines inclusive) and name it, for example, **certificate.pem**. Something like this:

``` 
-----BEGIN CERTIFICATE-----
MIIGMDCCBRigAwIBAgIKGpFzcgADAABKojANBgkqhkiG9w0BAQUFADA/MRQwEgYK
CZImiZPyLGQBGRYEaW5ldDESMBAGCgmSJomT8ixkARkWAmhpMRMwEQYDVQQDEwpJ
U1NVRUNBVElEMB4XDTE1MDYwMzA2MTY1NFoXDTE3MDYwMjA2MTY1NFowZzELMAkG
A1UEBhMCRVMxCzAJBgNVBAgTAk1BMQ8wDQYDVQQHEwZNYWRyaWQxEzARBgNVBAoT
ClRlbGVmb25pY2ExDDAKBgNVBAsTA1RpRDEXMBUGA1UEAxMOcGRpaHViLmhpLmlu
Oi8vb3JjYXRpZC5oaS5pbmV0L29jc3AwJgYIKwYBBQUHMAGGGmh0dHA6Ly9vcmNh
dGlkLnRpZC5lcy9vY3NwMFUGCCsGAQUFBzAChklodHRwOi8vaXNzdWVjYXRpZC5o
aS5pbmV0L0NlcnRFbnJvbGwvaXNzdWVjYXRpZC5oaS5pbmV0X0lTU1VFQ0FUSUQo
MykuY3J0MFIGCCsGAQUFBzAChkZodHRwOi8vb3JjYXRpZC5oaS5pbmV0L0NlcnRF
bnJvbGwvaXNzdWVjYXRpZC5oaS5pbmV0X0lTU1VFQ0FUSUQoMykuY3J0MFEGCCsG
AQUFBzAChkVodHRwOi8vb3JjYXRpZC50aWQuZXMvQ2VydEVucm9sbC9pc3N1ZWNh
dGlkLmhpLmluZXRfSVNTVUVDQVRJRCgzKS5jcnQwIQYJKwYBBAGCNxQCBBQeEgBX
AGUAYgBTAGUAcgB2AGUAcjANBgkqhkiG9w0BAQUFAAOCAQEALn2uGPE5cuOsDVWC
NJLJbFFH/+JwVWJjD7+oLg6z29fB17FJUdh7RgcV8ObPItvbqg2B5v773WSXRNp3
LQ3pRxefbwZdg2dbKTkcU9HSIX/WZHUUIFYGtXeQq7+9OJ5UY+Ta/Wg7ljKeLb3+
7gzyjDhBsipu/oXRMXInU3CppIoR802shofl4rXCoOTQNz9suQYISkLJJd2UBxsn
v1ytyPhqpdFr6zdsr2p/jFcxvj/uKYtMp9xeenHS8Nm/6y0sfPnzNpGTbxAhAoD0
jPIqRzg+X5Wo/A2bcS0hRkLWuQAWSnIJRUT1v+Bm62iB/C/OLR3zFRuxy+NwqHH5
q8+fMA==
-----END CERTIFICATE-----
```

If you don't have `openssl` or you don't want to install it you can Firefox to download the Certificate. Navigate to your server, click the _Lock_ in the address bar to see the information about the secure connection, click _More Information_ button, click _See Certificate_, go to _Details_ tab and click to export to save the file, name it as **certificate.pem** for example.

### Trusting the self-signed certificate

Copy/move the file to the folder you wish (personally, I prefer to create a hidden folder called `~/.git-certs/`) and store it there. Now, only left telling to [Git][git] to trust the certificate:

```
git config --global http.sslCAInfo ~/.git-certs/certificate.pem`
```

That's it, [Git][git] won't verify the SSL connection **ONLY** in the server what you need :)


If you have any questions or comments, please post them below.
If you liked this post, you can
<a href="https://twitter.com/intent/tweet?url=http://arturogutierrez.github.io{{ page.url }}&text={{ page.title }}&via={{ site.twitter_username }}" 
   target="_blank">
  share it with your followers</a> 
or 
<a href="https://twitter.com/{{ site.twitter_username }}">
  follow me on Twitter</a>!

<a href="https://twitter.com/share" class="twitter-share-button" data-url="http://arturogutierrez.github.io{{ page.url }}" data-via="{{ site.twitter_username }}" data-size="large">Tweet</a>

<!-- Put this just before the closing body tag -->
<script>!function(d,s,id){var js,fjs=d.getElementsByTagName(s)[0];if(!d.getElementById(id)){js=d.createElement(s);js.id=id;js.src="//platform.twitter.com/widgets.js";fjs.parentNode.insertBefore(js,fjs);}}(document,"script","twitter-wjs");</script>


[git]: https://git-scm.com
[pods]: https://cocoapods.org