+++
author = "hanchau"
title = "Gearman to the Rescue!"
date = "2020-03-01"
description = "Setup of the Gearman."
tags = [
    "gearman",
    "logs processor",
    "python",
    "redis",
]
categories = [
    "posts",
    "tool",
]
+++

This article offers a setup of [Gearman](https://github.com/hanchau/gearman_to_the_rescue) for Parallel/Distributed processing.
<!--more-->

### An elegant and generic application framework for parallel and distributed processing.

#### A web cast to get a good idea about the architecture of the Gearman.
---

{{< youtube YoAQ7Zsl9bE >}}

<br>


#### Initial Setup

1. Setup gearadmin and gearman with.
```
  $ brew install gearmand
  or
  $ apt install gearman-job-server
```
2. Setup [gearman job server](https://www.richardsumilang.com/server/gearman/install-gearman-on-os-x/) for mac.


3. You can get the [gearman python package](https://github.com/hanchau/gearman_to_the_rescue/tree/feature/arch/dependencies/gearman).

4. Copy the gearman package as shown and install the requirements.
```
  $ source /path/to/env
  $ python -m site
  $ cp -r <path/to/gearman/package/dir> </path/to/env>/lib/python3.8/site-packages/
  $ pip install -r requirements.txt
```

5. To check the proper installation do

```
  $ gearadmin --server-version
  $ gearadmin --status
```

6. To check the proper installation of the python gearman package do
```
  $ python -c "import gearman"
```
`if no error is thrown then you are good to go.`
