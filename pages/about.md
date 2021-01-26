---
layout: page
title: About
description: Information Security
keywords: Isab
comments: true
menu: 关于
permalink: /about/
---

认识自己的过程总是伴随着一些不可思议，这种不可思议主要表现在前后的巨大反差上，比方说在做一件事情之前，我会觉得自己道理都懂，一旦出手必定手到擒来，但是一旦开始做了，却发现自己举步维艰，所知甚少。所以时常有人说，要知道自己是个什么样的人，就从此刻起做一件你此时认为不可能完成的事情。

## 联系

<ul>
{% for website in site.data.social %}
<li>{{website.sitename }}：<a href="{{ website.url }}" target="_blank">@{{ website.name }}</a></li>
{% endfor %}
</ul>


## Skill Keywords

{% for skill in site.data.skills %}
### {{ skill.name }}
<div class="btn-inline">
{% for keyword in skill.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
