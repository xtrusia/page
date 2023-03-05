---
layout: post
title: Build MAAS from source
nav_order: 2
reflowMarkdown.preferredLineLength: 60
---

#This is short note for my work

Although most of process are covered by doc.

{% highlight shell %}

lxd init --auto

git clone https://github.com/maas/maas

git submodule update --init --recursive

make snap

{% endhighlight %}