---
title:  使用 async 控制并发数量
date: 2018-09-24 23:45:51
categories: node
tags:
 - async
 - 并发
---

```js
const eventproxy = require('eventproxy')
const superagent = require('superagent')
const cheerio = require('cheerio')
const url = require('url')
const async = require('async')
const ep = new eventproxy()
const cnodeUrl = 'https://cnodejs.org'
const topicUrls = [];
superagent.get(cnodeUrl)
  .end(function (err, res) {
    if (err) {
      return console.error(error)
    }
    const topicUrls = [];
    const $ = cheerio.load(res.text);
    $('#topic_list .topic_title').each((index, item) => {
      const href = url.resolve(cnodeUrl, $(item).attr('href'))
      topicUrls.push(href)
    })
    console.log(topicUrls.length)
    let count = 0;
    async.mapLimit(topicUrls, 6, function (url, callback) {
      count++
      console.log(`现在的并发数${count}`)
      superagent.get(url)
        .end(function (err, res) {
          if (err) return console.log(`fetch faild ${err.status}`)
          count--
          callback(null, [url, res.text])
        })
    }, function (err, result) {
      if (err) return console.log(err)
      result = result.map(item => {
        const $ = cheerio.load(item[1]);
        return ({
          title:$('.topic_full_title').text().trim(),
          url:item[0],
          comment1:$('.reply_content').eq(0).text().trim(),
        })
      })
      console.log(result)
      console.log(result.length)
    })
  })
```