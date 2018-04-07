---
title: 在Jekyll上使用LeanCloud統計訪問人數
tags: Jekyll LeanCloud
categories: Blog
---

* TOC
  {:toc}

使用`Jekyll` + `Github Pages`建立 blog 非常方便快速，網路上也有很多教學從安裝環境、Jekyll 架構、修改方式到最後發布自己的 blog，其中很方便的是 Jekyll 支援許多第三方套件，讓我們不用自己刻的半死，例如:

* **[Swiftype]{:target="_blank"}**

  提供 blog 內使用關鍵字尋文章的功能

* **[Disqus]{:target="_blank"}**

  可以針對文章使用評論、推薦的功能

* **[LeanCloud]{:target="_blank"}**

  這是這次要介紹的套件，可以實現對部落格的進行統計訪問人數

LeanCloud 提供了雲端資料儲存、資料分析、即時訊息推播...等服務，是一個強大的後端平台，那我這次會使用的為它的儲存庫、還有其 API 的功能，能讓我們把資料從 blog 往儲存庫送。這次最終要實現的為在文章上顯示訪問次數，並且在進入文章時能夠增加次數並儲存。

# 註冊 LeanCloud 及建立應用

首先要先註冊自己的 LeanCloud 帳號， 並且建立應用，完成後在設置底下會拿得一組`App ID`及`App Key`，這會在後續設定 blog 時使用。

![createApp][createapp]

![register][register]

# 建立 Class

`Class`就是我們會拿來放訪問人數資料的地方，名稱後續會在 blog 撰寫 API 介接時使用，所以最好是有意義的命名，權限則採用預設的限制寫入。

![createApp][createclass]

# \_config.yml 檔設定

```yml
leancloud:
  app_id: app_id
  app_key: app_key
```

這樣我們後續就可以透過 site.leancloud 取得這個資訊來使用。

# 存取 LeanCloud 的 JS 程式碼

在`_includes`目錄下新增一隻 leancloud-analytics.html 檔案，而這支檔案是作為存取 LeanCloud 行為的 javascript 程式碼。

```html
<script src="https://code.jquery.com/jquery-3.2.0.min.js"></script>
<script src="https://cdn1.lncld.net/static/js/av-core-mini-0.6.1.js"></script>
<script>AV.initialize("{{site.leancloud.app_id}}", "{{site.leancloud.app_key}}");</script>
<script>
    //顯示訪問數
    function showHitCount(Counter) {
        var query = new AV.Query(Counter);
        var entries = [];
        var $visitors = $(".show_leancloud_visitors");
        $visitors.each(function () {
            entries.push( $(this).attr("id").trim() );
        });
        query.containedIn('url', entries);
        query.find()
                .done(function (results) {
                    console.log("results",results);
                    var COUNT_CONTAINER_REF = '.leancloud-visitors-count';
                    if (results.length === 0) {
                        $visitors.find(COUNT_CONTAINER_REF).text(0);
                        return;
                    }
                    for (var i = 0; i < results.length; i++) {
                        var item = results[i];
                        var url = item.get('url');
                        var hits = item.get('hits');
                        var element = document.getElementById(url);
                        $(element).find(COUNT_CONTAINER_REF).text(hits);
                    }
                    for(var i = 0; i < entries.length; i++) {
                        var url = entries[i];
                        var element = document.getElementById(url);
                        var countSpan = $(element).find(COUNT_CONTAINER_REF);
                        if( countSpan.text() == '') {
                            countSpan.text(0);
                        }
                    }
                })
                .fail(function (object, error) {
                    console.log("Error: " + error.code + " " + error.message);
                });
    }
    //增加訪問數
    function addCount(Counter) {
        var $visitors = $(".add_leancloud_visitors");
        var url = $visitors.attr('id').trim();
        var title = $visitors.attr('data-flag-title').trim();
        var query = new AV.Query(Counter);
        query.equalTo("url", url);
        query.find({
            success: function(results) {
                if (results.length > 0) {
                    var counter = results[0];
                    counter.fetchWhenSave(true);
                    counter.increment("hits");
                    counter.save(null, {
                        success: function(counter) {
                            var $element = $(document.getElementById(url));
                            $element.find('.leancloud-visitors-count').text(counter.get('hits'));
                        },
                        error: function(counter, error) {
                            console.log('Failed to save Visitor num, with error message: ' + error.message);
                        }
                    });
                } else {
                    var newcounter = new Counter();
                    var acl = new AV.ACL();
                    acl.setPublicReadAccess(true);
                    acl.setPublicWriteAccess(true);
                    newcounter.setACL(acl);
                    newcounter.set("title", title);
                    newcounter.set("url", url);
                    newcounter.set("hits", 1);
                    newcounter.save(null, {
                        success: function(newcounter) {
                            var $element = $(document.getElementById(url));
                            $element.find('.leancloud-visitors-count').text(newcounter.get('hits'));
                        },
                        error: function(newcounter, error) {
                            console.log('Failed to create');
                        }
                    });
                }
            },
            error: function(error) {
                console.log('Error:' + error.code + " " + error.message);
            }
        });
    }
    $(function() {
        var Counter = AV.Object.extend("Counter");
        if ($('.add_leancloud_visitors').length >= 1) {
            // add 1 for counter
            addCount(Counter);
        } else if ($('.show_leancloud_visitors').length >= 1){
            // show counter
            showHitCount(Counter);
        }
    });
</script>
```

然後在\_layouts 目錄底下的 default.html(或其他共用的模板)內載入上面的 leancloud-analytics，提供所有頁面可以存取。

![include-leancloud-analytics][include-leancloud-analytics]

# 顯示訪問數

在 post.html 內要顯示訪問數的位置新增如下 html

```html
<span id="{{ page.url }}" class="add_leancloud_visitors" data-flag-title="{{ page.title }}">
    <span class="octicon octicon-eye"></span>
    <span class="old-visitors-count" style="display: none;"></span>
    <span class="leancloud-visitors-count"></span>
    <span class="post-meta-item-text">次瀏覽</span>
</span>
```

這邊要注意的是 span 的 class 名稱是 add_leancloud_visitors，因為當閱讀者進入到文章內顯示訪問數以外，系統也需要同步新增訪問數，所以需調用新增訪問數的函式，如只是單純顯示，像是在外面的文章列表不需要自動增加，則可以修改為 show_leancloud_visitors。

而 id 拿到的 url 則是文章的位址，會將此 url 紀錄在 LeanCloud 上，下次有讀者進入到同一個文章時，就新增同一個 url 的訪問數。

![result][result]

成功後在 LeanCloud 上我所新增的 class 下面，就會開始記錄被訪問的資料，title、url、hits 是我要關注的資訊，當然如果有其他資訊是需要被關注的，可以於 JS 程式碼新增欄位和值。
![counterData][counterdata]

Try it!

使用 Jekyll 建立部落格，怎麼可以沒有人數統計

[swiftype]: https://swiftype.com/
[disqus]: https://disqus.com/
[leancloud]: https://leancloud.cn/

[register]: {{"/2018-04-04-leancloud-In-Jekyll/register.png" | prepend: site.imgrepo }}
[createApp]: {{"/2018-04-04-leancloud-In-Jekyll/createApp.png" | prepend: site.imgrepo }}
[createClass]: {{"/2018-04-04-leancloud-In-Jekyll/createClass.png" | prepend: site.imgrepo }}
[include-leancloud-analytics]: {{"/2018-04-04-leancloud-In-Jekyll/include-leancloud-analytics.png" | prepend: site.imgrepo }}
[result]: {{"/2018-04-04-leancloud-In-Jekyll/result.png" | prepend: site.imgrepo }}
[counterData]: {{"/2018-04-04-leancloud-In-Jekyll/counterData.png" | prepend: site.imgrepo }}
