---
published: true
title: curl downloads from ownCloud public links
layout: post
---
On a public ownCloud page take the link from the "Download" button on the top right side. Then:

```bash
curl -J -O https://owncloud.address.com/s/jdjdsjiawn/download
```