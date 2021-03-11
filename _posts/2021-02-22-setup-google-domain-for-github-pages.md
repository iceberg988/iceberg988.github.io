---
layout:     post
title:      "Setup google domain for github pages"
subtitle:   ""
date:       2021-02-22 14:00:00
author:     "Iceberg"
header-style: text
catalog:    true
tags:
  - WebApp
---

1. In the GitHub repository settings, specify your Google domain name to serve the GitHub pages.
   ![](/assets/images/github-setting.png)

   ![](/assets/images/github-custom-domain.png)

2. Go to Google Domains DNS view, create an [A record](https://docs.github.com/en/github/working-with-github-pages/managing-a-custom-domain-for-your-github-pages-site#configuring-a-records-with-your-dns-provider) and CNAME entry to point your Google domain at your web server. 

   ![](/assets/images/google-domain-dns.png)

   ![](/assets/images/google-domain-cname.png)

   To verify the DNS record configured correctly, use the dig command to confirm the results match the IP addresses for GitHub Pages above.

   ```shell
   $ dig flamingbytes.com +noall +answer
   ; <<>> DiG 9.10.6 <<>> flamingbytes.com +noall +answer
   ;; global options: +cmd
   flamingbytes.com.	3600	IN	A	185.199.110.153
   flamingbytes.com.	3600	IN	A	185.199.108.153 
   flamingbytes.com.	3600	IN	A	185.199.109.153
   flamingbytes.com.	3600	IN	A	185.199.111.153
   ```   

   To verify the CNAME record configured corretly, use the dig command to check as the following.

   ```shell
   $ dig www.flamingbytes.com +nostats +nocomments +nocmd
   ; <<>> DiG 9.10.6 <<>> www.flamingbytes.com +nostats +nocomments +nocmd
   ;; global options: +cmd
   ;www.flamingbytes.com.		IN	A
   www.flamingbytes.com.	3406	IN	CNAME	iceberg988.github.io.
   iceberg988.github.io.	3600	IN	A	185.199.108.153
   iceberg988.github.io.	3600	IN	A	185.199.110.153
   iceberg988.github.io.	3600	IN	A	185.199.109.153
   iceberg988.github.io.	3600	IN	A	185.199.111.153
   ```

4. Go back to GitHub repository setting page, clear and re-enter your Google domain name to enable https.
 
   ![](/assets/images/github-setting-https.png)

5. In about 15 minutes, the GitHub pages will be published in your Google Domain.

   ![](/assets/images/github-setting-publish.png)

   You may see the following page before the website can be accessed successfully. You just wait for 15 minutes to get it successfully published.

   ![](/assets/images/github-certificate-issue.png)
   
