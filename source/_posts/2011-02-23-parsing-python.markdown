---
author: admin
comments: true
date: 2011-02-23 15:16:04+00:00
layout: post
slug: parsing-python
title: Парсинг на Python или как скачать обои в большом разрешении
wordpress_id: 658
categories:
- python
- парсеры
tags:
- images
- parser
- python
---

[![python-parser](http://vredniy.ru/wp-content/uploads/2011/02/python-parser-150x150.png)](http://vredniy.ru/wp-content/uploads/2011/02/python-parser.png)
В сегодняшней заметки мы с вами получим порядка 1500+ в высоком разрешении, позаимствовав из с некоторого сайта путем парсинга html страниц.
<!-- more -->
Немного поковырялся я в Python (больно понравился мне его синтаксис) и чтобы закрепить полученные начальные знания решил на нем небольшую прикладную задачку, которая сводится к взятию с сайта b000.ru всех картинок. Чуть ниже исходный код:
[cc lang="python"]
# -*- coding: utf-8 -*-
''' 
image parser 
(C) vredniy
vredniy.ru
'''
from urllib2 import Request, urlopen, URLError, HTTPError
from urllib import urlretrieve
import os
import sys
import re
import time

def doRoutine(url):
     req = Request(url)
     try: response = urlopen(req)

except HTTPError, e:
     print e.code
     return 0
except URLError, e:
     print e.reason
     return 0
else:
     print url
     data = response.read()

m = re.search('

[\w\W]*?

# (.*?)

[\W\w]+?[<img[\w\W]*?alt="(.*?)"', data, re.IGNORECASE & re.UNICODE);
if m:
     downloadImage(m.group(2), m.group(3))


def downloadImage(imageUrl, imageName):
     urlretrieve(imageUrl, './images/' + imageName + '.jpg')
     print imageName + ' скачано'

def main(url):
     counter = 1
     while (counter < 1900):
     counter = counter + 1
     doRoutine("%s%d" % (url, counter))

if __name__ == "__main__":
url = 'http://b000.ru/view/'
main(url)
[/cc]
И в папочке images сохраняются все обои, правда занимает это продолжительное время :)
