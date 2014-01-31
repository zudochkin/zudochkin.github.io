---
author: admin
comments: true
date: 2010-07-09 08:57:49+00:00
layout: post
slug: sitemap-modx
title: Карта сайта MODx - короткая заметка
wordpress_id: 535
categories:
- cms
tags:
- cms
- modx
---

[![modx-sitemap](http://vredniy.ru/wp-content/uploads/2010/07/modx-logo-evolution-150x114.png)](http://vredniy.ru/wp-content/uploads/2010/07/modx-logo-evolution.png)Понадобилось вставить карту сайта на CMS, с которой почти не знаком. После нескольких минут гугления нашлось решение.
Создается обычная страница, в html код которой помещается фраза [cc lang="php"][[Wayfinder? &startId;=`0`]][/cc].
Теперь на этой странице в иерархическом виде отображется карта сайта. Страницы, которые не видимы в меню, не отображаются.
