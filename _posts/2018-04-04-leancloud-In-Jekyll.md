---
title: 在Jekyll上使用LeanCloud統計點擊人數
tags: Jekyll LeanCloud
categories: Blog
---

* TOC 
{:toc}

使用`Jekyll` + `Github Pages`建立blog非常方便快速，網路上也有很多教學從安裝環境、Jekyll架構、修改方式到最後發布自己的blog，其中很方便的是Jekyll支援許多第三方套件，讓我們不用自己刻的半死，例如:

- **[Swiftype]**

    提供blog內使用關鍵字尋文章的功能

- **[Disqus]**

    可以針對文章使用評論、推薦的功能

- **[LeanCloud]**

    這是這次要介紹的套件，可以實現對部落格的進行統計點擊人數

LeanCloud提供了雲端資料儲存、資料分析、即時訊息推播...等服務，是一個強大的後端平台，那我這次會使用的為它的儲存庫、還有其API的功能，能讓我們把資料從blog往儲存庫送。
這次最終要實現的為在文章上顯示點擊次數，並且在進入文章時能夠增加次數並儲存。

# 註冊LeanCloud及建立應用
首先要先註冊自己的LeanCloud帳號， 並且建立應用，完成後在設置底下會拿得一組`App ID`及`App Key`，這會在後續設定blog時使用。

![createApp][createApp]

![register][register]

# 建立Class
`Class`就是我們會拿來放點擊人數資料的地方，名稱後續會在blog撰寫API介接時使用，所以最好是有意義的命名，權限則採用預設的限制寫入。

![createApp][createClass]

# _config.yml檔設定

~~~yml
leancloud:
  app_id: app_id
  app_key: app_key
~~~

這樣我們後續就可以透過site.leancloud取得這個資訊來使用。


# 存取LeanCloud的JS程式碼
在`_includes`目錄下新增一隻leancloud-analytics.html檔案，而這支檔案是作為存取LeanCloud行為的javascript程式碼。

~~~html
<script src="https://code.jquery.com/jquery-3.2.0.min.js"></script>
<script src="https://cdn1.lncld.net/static/js/av-core-mini-0.6.1.js"></script>
<script>AV.initialize("{{site.leancloud.app_id}}", "{{site.leancloud.app_key}}");</script>
<script>
    //顯示點擊數
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
    //增加點擊數
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
        if ($('.add_leancloud_visitors').length == 1) {
            // add 1 for counter
            addCount(Counter);
        } else if ($('.show_leancloud_visitors').length == 1){
            // show counter
            showHitCount(Counter);
        }
    });
</script>
~~~

然後在_layouts目錄底下的default.html(或其他共用的模板)內載入上面的leancloud-analytics，提供所有頁面可以存取。

![include-leancloud-analytics][include-leancloud-analytics]

# 顯示點擊數

在post.html內要顯示點擊數的位置新增如下html

~~~html
<span id="{{ page.url }}" class="add_leancloud_visitors" data-flag-title="{{ page.title }}">
    <span class="octicon octicon-eye"></span>
    <span class="old-visitors-count" style="display: none;"></span>
    <span class="leancloud-visitors-count"></span>
    <span class="post-meta-item-text">次瀏覽</span>
</span>
~~~

這邊要注意的是span的class名稱是add_leancloud_visitors，因為當閱讀者進入到文章內顯示點擊數以外，系統也需要同步新增點擊數，所以需調用新增點擊數的函式，如只是單純顯示，像是在外面的文章列表不需要自動增加，則可以修改為show_leancloud_visitors。

而id拿到的url則是文章的位址，會將此url紀錄在LeanCloud上，下次有讀者進入到同一個文章時，就新增同一個url的點擊數。

![result][result]

成功後在LeanCloud上我所新增的class下面，就會開始記錄被點擊的資料，title、url、hits是我要關注的資訊，當然如果有其他資訊是需要被關注的，可以於JS程式碼新增欄位和值。
![counterData][counterData]

Try it!

使用Jekyll建立部落格，怎麼可以沒有人數統計


[Swiftype]: https://swiftype.com/
[Disqus]: https://disqus.com/
[LeanCloud]: https://leancloud.cn/

[register]: {{"/2018-04-04-leancloud-In-Jekyll/register.png" | prepend: site.imgrepo }}
[createApp]: {{"/2018-04-04-leancloud-In-Jekyll/createApp.png" | prepend: site.imgrepo }}
[createClass]: {{"/2018-04-04-leancloud-In-Jekyll/createClass.png" | prepend: site.imgrepo }}
[include-leancloud-analytics]: {{"/2018-04-04-leancloud-In-Jekyll/include-leancloud-analytics.png" | prepend: site.imgrepo }}
[result]: {{"/2018-04-04-leancloud-In-Jekyll/result.png" | prepend: site.imgrepo }}
[counterData]: {{"/2018-04-04-leancloud-In-Jekyll/counterData.png" | prepend: site.imgrepo }}