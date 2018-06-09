---
title: 在Django上進行建立及測試404/500 Error Page
tags: Django
categories: Python
---

* TOC
{:toc}

本文章將會說明如何使用Django建立404/500 error page及他們的測試方法，因為Django在DEBUG為True和False時都有各自的麻煩之處，會導致開發者很難驗證這功能的正確性。

# 建立 404/500 Error Page

首先要在 `view.py` 檔內新增 `handler404` 及 `handler500` 函式，並各自導到屬於他們的畫面，response的status_code則代表系統應該要回應的 `http status` 是什麼。

`handler404` 及 `handler500` 是Djano已經定義好的功能，所以只要做到這裡，error page基本上已經是完成了

```python
def handler404(request):
    response = render(request, 'error_page/404-error-page.html')
    response.status_code = 404
    return response


def handler500(request):
    response = render(request, 'error_page/500-error-page.html')
    response.status_code = 500
    return response
```

# DEBUG=True時的Error Page測試

頁面設計好了，對應的處理也好了，總要來看一下是不是符合心中所期待的樣貌，在DEBUG=True情況下，我們可以在 `urls.py` 內建立路徑直接來看頁面是否正確，說穿了就是直接在網址後面接上404或500就導到頁面來檢視。

```python
urlpatterns = [ 
    url(r'^404/$', handler404),
    url(r'^500/$', handler500),
]
```

看到這裡一定覺得很奇怪，為什麼DEBUG=True時不能直接進行測試，像是亂輸入網址來顯示404 error page，是因為在debug=True時Django還是必須要導到debug頁面才真正幫助開發者。

# DEBUG=False時的Error Page測試

那好吧，我們總可以設定為DEBUG=False來進行測試，但會發現這時候又會遇到另一個讓心情很不好的問題，Django在DEBUG=False時網站上的static files會無法正確載入，這樣會導致可以成功導到error page，但是客倌啊，通通沒有樣式!

這時的解決方式為使用以下方式來運行網站，一切問題都解決了! (早說嘛)

```python
python manage.py runserver --insecure
```

# Error 500 的測試

Error 404只要亂輸入網址就好，很容易測試，但500為 `Internal Server Error`，當然是要在我們不預期的情況下發生，但如果是為了測試，則可以故意建立一個會出錯的函式。

以下為會出錯的範例，然後記得在urls.py加上路徑，這樣我們就可以直接透過路徑來測試它了。

```python
def test_handler500(request):
    a = 1/0
    print(a)
    return request
```

Try it!
