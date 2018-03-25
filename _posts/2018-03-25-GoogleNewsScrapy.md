---
title: 使用Python爬抓取Google新聞
tags: Scrapy BeautifulSoup
categories: Python
---

最近有需求要做一隻Python爬蟲來抓取新聞，與其到各大網路新聞去抓，乾脆直接把腦筋動到Google上，以Google抓新聞的管道我看到有兩個，一個是[Google search的新聞頁籤][GoogleSearchNews]，另一個則是[Google新聞][GoogleNews]。

Google新聞平台上沒有更多篩條件，抓下來的資料在時間還有排序上只能完全依照google提供的(汗)。所以我最後選擇用Google Search的新聞做為資料來源。

不過使用Goolgle Search的新聞，發現在網頁點選右鍵檢視網頁原始碼所看到的html會與實際爬蟲抓到的不一樣，看起來Google針對這部分有把html標籤、id、class等變化過，所以在處理前必須要先了解一下爬蟲抓到的資料結構。

- **右鍵檢視網頁原始碼看到的html**

~~~html
<div class="gG0TJc">
    <h3 class="r dO0Ag">
        <a class="l lLrAF" href="https://www.udn.com/news/story/7002/3049761" ping="/url?sa=t&amp;source=web&amp;rct=j&amp;url=https://www.udn.com/news/story/7002/3049761&amp;ved=0ahUKEwjO7r-R14XaAhUFp5QKHcmXDTAQqQIIJSgAMAA">
            <em>NBA</em>／厄文將動左膝手術回歸時間未定
        </a>
    </h3>
    <div class="slp">
        <span class="xQ82C e8fRJf">udn 聯合新聞網</span>
        <span class="v0c3xd">-</span>
        <span class="f nsa fwzPFf">10 小時前</span>
    </div>
    <div class="st">
        厄文（左）將會接受左膝微創手術，藉此減輕膝蓋的疼痛感，復出時間未定。
        <em>NBA</em> 分享. facebook. 根據塞爾蒂克的官方消息，當家球星厄文（Kyrie Irving）將會接受左膝微創手術，藉此減輕膝蓋的疼痛感。不過厄文會何時復出，恐怕要等手術完成後才能有確切時間。 厄文本季一直受到左膝傷困擾，單是三月份就缺席了6場比賽，近期已經&nbsp;...
    </div>
</div>
~~~


- **實際爬蟲抓到的html**

~~~html
<div class="g">
    <table>
        <tr>
            <td valign="top" style="width:516px">
                <h3 class="r">
                    <a href="/url?q=https://technews.tw/2018/03/16/the-leader-of-5g-tech-ericsson-will-help-chonghwa-telecom-to-5g-network-possible-in-the-end-of-2018/&amp;sa=U&amp;ved=0ahUKEwiKoNqI4ILaAhXJNJQKHTaGDDYQqQIIJygAMAM&amp;usg=AOvVaw2NCPP3YIn2EJTFLFIrNRZE">5G 王者愛立信將助
                        <b>中華電信</b>年底推5G 示範網路
                    </a>
                </h3>
                <div class="slp">
                    <span class="f">科技新報 TechNews - 2018年3月16日</span>
                </div>
                <div class="st">我們聽到很多關於5G 技術或是創新應用，也很期待5G 帶來的更快速，低延遲的連線，驅動很多以前不敢想像的應用。如今在台灣，5G 電信網路總算要來，根據供應全台電信業者所需通訊技術方案廠商愛立信所言，
                    <b>中華電信</b>最快今年(2018) 底推出5G 行動網路服務。
                </div>
                <a href="/url?q=https://newtalk.tw/news/view/2018-03-16/117637&amp;sa=U&amp;ved=0ahUKEwiKoNqI4ILaAhXJNJQKHTaGDDYQ-AsIKSgAMAM&amp;usg=AOvVaw0PchLFBASmlwIlrMEzMClA">5G台灣用的到
                    <b>中華電信</b>最快年底推出
                </a>
                <span class="f">新頭殼</span>
                <br>
                <a href="/url?q=http://www.chinatimes.com/newspapers/20180317001058-260210&amp;sa=U&amp;ved=0ahUKEwiKoNqI4ILaAhXJNJQKHTaGDDYQ-AsIKigBMAM&amp;usg=AOvVaw21H3Of9kl4N0wMrekocU64">愛立信攜手
                    <b>中華</b>電年底推出5G示範場域
                </a>
                <span class="f">中時電子報 (新聞發布)</span>
                <br>
            </td>
            <td valign="top" style="padding:2px 0 0 8px;font-size:77%;text-align:center">
                <a href="/url?q=https://technews.tw/2018/03/16/the-leader-of-5g-tech-ericsson-will-help-chonghwa-telecom-to-5g-network-possible-in-the-end-of-2018/&amp;sa=U&amp;ved=0ahUKEwiKoNqI4ILaAhXJNJQKHTaGDDYQpwIIKzAD&amp;usg=AOvVaw2qTG0mHwOD0OfJmmFsrFYI">
                    <img class="th" src="https://encrypted-tbn2.gstatic.com/images?q=tbn:ANd9GcSrMsQNX2LyHSetooCdTMB0_1t5sOqPedGKDDCqww8dvG2bsvCUzjrO13Ku801WhK1JleQpVGQ"
                        border="1">
                </a>
                <div class="f">科技新報 TechNews</div>
            </td>
        </tr>
    </table>
</div>
~~~

- **可運作的程式碼**

~~~python
import requests
from bs4 import BeautifulSoup
from urllib import quote
import urlparse

search_list = []
keyword = quote('"柯文哲"'.encode('utf8'))
res=requests.get("https://www.google.com.tw/search?num=50&q="+keyword+"&oq="+keyword+"&dcr=0&tbm=nws&source=lnt&tbs=qdr:d")

# 關鍵字多加一個雙引號是精準搜尋
# num: 一次最多要request的筆數, 可減少切換頁面的動作
# tbs: 資料時間, hour(qdr:h), day(qdr:d), week(qdr:w), month(qdr:m), year(qdr:w)

if res.status_code == 200:
    content = res.content
    soup = BeautifulSoup(content, "html.parser")

    items = soup.findAll("div", {"class": "g"})
    for item in items:
        # title
        news_title = item.find("h3", {"class": "r"}).find("a").text

        # url
        href = item.find("h3", {"class": "r"}).find("a").get("href")
        parsed = urlparse.urlparse(href)
        news_link = urlparse.parse_qs(parsed.query)['q'][0]

        # content
        news_text = item.find("div", {"class": "st"}).text

        # source
        news_source = item.find("span", {"class": "f"}).text.split('-')
        news_from = news_source[0]
        time_created = str(news_source[1])

        # add item into json object
        search_list.append({
            "news_title": news_title,
            "news_link": news_link,
            "news_text": news_text,
            "news_from": news_from,
            "time_created": time_created
        })
~~~

以下為幾個關鍵
- **url encode**
~~~python
keyword = quote('"柯文哲"'.encode('utf8'))
~~~
中文字放到網址內需先進行編碼轉換，只要利用 `urllib` 的 `quote` 即可輕鬆轉換。

- **findAll and find**
~~~python
items = soup.findAll("div", {"class": "g"})
~~~
~~~python
news_title = item.find("h3", {"class": "r"}).find("a").text
~~~
指定要查詢的標籤及其要篩選的屬性和條件， `find` 和 `findAll` 的差別在於前者為回傳一筆，後者為取得多筆資料'，而使用 `.text` 可以取得其內容。

- **get url query**
~~~python
parsed = urlparse.urlparse(href)
news_link = urlparse.parse_qs(parsed.query)['q'][0]
~~~
使用 `urlparse` 解析url 以及 `parse_qs` 函數擷取query參數，拿到的結果會是一個陣列所以要指定index來取得值。

初學Python，盡量把學習的一些紀錄記下來，希望對於學習爬蟲及需要爬Google新聞的人有一些幫助。


[GoogleNews]: https://news.google.com/news/search/section/q/NBA/NBA?hl=zh-tw&gl=TW&ned=zh-tw_tw
[GoogleSearchNews]: https://www.google.com.tw/search?q=NBA&num=50&dcr=0&source=lnms&tbm=nws&sa=X&ved=0ahUKEwiGwZaj0YXaAhUGkpQKHbRIBmcQ_AUICigB&biw=1163&bih=559