---
title: 在Windows上使用VS Code開發Django
tags: VSCode Python
categories: Tools
---

過往開發Pythone專案時，我都是使用`Pycharm`作為開發工具，專門為了開發Python的編輯器，因而不用搞太多環境設定的問題，但其實要開發Django的話必須要使用專業版本(雖然有學生帳號就可以下載到)，且近期使用下來發現效能不佳，所以考慮轉換到自己開發前端所熟悉的VS code，強大且眾多的套件可以使用才是王道啊。

廢話部囉嗦，請先[下載 VS Code][downloadVSCode] 然後開始建立開發環境吧。

# 設定環境變數

本機 -->　內容　--> 進階設定　-->　環境變數，在Path變數內新增以下兩個路徑，請依據您Python安裝的路徑及名稱而定：

* **D:\Python27**
* **D:\Python27\Scripts**

# 安裝VS Code套件

* **Python**
* **Django**
* **Django Template**

# 設定虛擬環境

* **使用 pip install virtualenv 安裝虛擬環境**
* **使用 virtualenv "ENV Name" 建立虛擬目錄**
* **到已建立的虛擬環境的目錄的 Scripts 目錄內，使用 activate.bat 啟動虛擬環境**
* **使用 pip install -r requirements.txt 安裝專案的模組，或 pip install "模組名稱--版本"**

# 建立 Djanog 專案

* **使用 django-admin startproject demo 建立django專案資料夾**
* **使用 python manage.py startapp web 新建Django App**

# 指定 Python Interpreter

* **使用 VS Code 開啟 Django 專案**
* **在 VS Code 中按下 Ctrl+Shift+P，輸入並選擇 Python: Select Interpreter**
* **將 Python Interpreter 指定為虛擬環境的 python**

最後將 Vs Code 的 debug 模式設定為 Django，然後按下 F5 即可!

[downloadVSCode]: https://code.visualstudio.com/Download
