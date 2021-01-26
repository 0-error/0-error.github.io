---
layout: page
title: About
description: Information Security
keywords: Isab
comments: true
menu: 关于
permalink: /about/
---

## 关于

没有绝对安全的系统，

因为系统是我装的。

我是 isab，信息安全从业者。

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
