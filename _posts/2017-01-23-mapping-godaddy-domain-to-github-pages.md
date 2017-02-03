---
layout: post
title: Mapping GoDaddy Domain to Github Pages
---

For the goal of developing my personal brand, I've spent all my money and effort on buying a 10-dollar domain and mapped it to my Github pages. In fact, I had struggled with this job myself for 1 hour until I got it done with the help of my friend Google. These are the steps.

1. Create your Github pages

   I'll assume you already have a website like `<yourUsename>.github.io`.

   > Don't know how to create it? Follow [this](https://pages.github.com).  
   > Don't have much time? Go with [Jekyll Now](https://github.com/barryclark/jekyll-now).

2. Create a CNAME file.

   Add your custom domain to that file e.g. "emeraldhieu.me" in my case.
   ![Create CNAME file]({{ site.baseurl }}/public/images/2017-01-23-mapping-godaddy/create-cname-file.png "Create CNAME file")

3. Configure GoDaddy

   Login GoDaddy. In "Domain Details", select "DNS Zone File".
   
   Create a record "@" that points to "192.30.252.153" in table "A (Host)".
   ![Create an A record]({{ site.baseurl }}/public/images/2017-01-23-mapping-godaddy/create-an-a-record.png "Create an A record")

   Create a record "www" that points to `<yourUsename>.github.io` in table "CName (Alias)"
   ![Create a CNAME record]({{ site.baseurl }}/public/images/2017-01-23-mapping-godaddy/create-a-cname-record.png "Create a CNAME record")

4. Wait for changes to be updated

	Check your work with this command  

   ```bash
   dig <yourDomain> +nostats +nocomments +nocmd
   ```

	![Wait for changes]({{ site.baseurl }}/public/images/2017-01-23-mapping-godaddy/wait-for-changes.png "Wait for changes")

DONE!