# Deploying_Django_To_Heroku_With_Docker

教你如何 Deploying Django To Heroku With Docker

* [Youtube Tutorial - Deploying Django To Heroku With Docker](https://youtu.be/2Pl6iALuJgQ)

## 簡介

之前已經教學過大家如何將 Django 以及 Flask 佈署到 Heroku 上了，如果你還沒看過或不懂，請先參考以下兩篇，

* [Deploying a Django App To Heroku Tutorial](https://github.com/twtrubiks/Deploying_Django_To_Heroku_Tutorial)

* [Deploying a Flask App To Heroku Tutorial](https://github.com/twtrubiks/Deploying-Flask-To-Heroku)

那今天我要更進階教大家使用 Docker 將 Django 佈署到 Heroku，如果你不懂 Docker，也請先閱讀以下文章

* [Docker 基本教學 - 從無到有 Docker-Beginners-Guide](https://github.com/twtrubiks/docker-tutorial)

這次使用 Docker 將 Django 佈署到 Heroku 上遇到很雷的事情 :cry:

讓我們看下去:smiley:

## 教學

請先確認你的電腦安裝了 [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli) ( 如果你有曾經部署過，應該都有安裝了 )，

另一個要安裝的是 [Docker](https://docs.docker.com/) ( 現在要下載 Docker 必須要登入，簡單去註冊一個吧 :wink:)。

以上都做好了之後，請先確認使否已經登入 Docker 以及 Heroku，

登入 Docker，

```cmd
docker login
```

登入 Heroku，

```cmd
heroku login
```

接著 [Logging in to the registry](https://devcenter.heroku.com/articles/container-registry-and-runtime#logging-in-to-the-registry)，執行以下指令，

```cmd
heroku container:login
```

( 這邊如果你是 windows 用戶請注意，請使用 **內建的命令提示字元** 執行，像我如果使用 cmder 執行會錯誤，很重要 :exclamation:)

( 另外，當你執行 `heroku login` 時，如果你已經登入過，然後再次執行時，請重新輸入一次，不要使用 Ctrl+C 中斷 ，不然也會導致錯誤 )

以上兩點很重要，我就是被這個雷到:sob:

我們先在本地端使用 docker 測試，確認測試沒問題，我們再部署到 Heroku 上，

我們使用的範例是我之前寫的這篇 [Docker 基本教學 - 從無到有 Docker-Beginners-Guide](https://github.com/twtrubiks/docker-tutorial)，

然後有一點要注意的是，Heroku 的佈署並不支援 docker-compose 的方式 :sob: ( 這個我們後面再進一步說明 )，

現在我們只使用一個 image ( 也可以說一個 container )。

先來看 [Dockerfile](https://github.com/twtrubiks/Deploying_Django_To_Heroku_With_Docker/blob/master/Dockerfile)，

```Dockerfile
FROM python:3.6.2
LABEL maintainer twtrubiks
ENV PYTHONUNBUFFERED 1
RUN mkdir /docker_api
WORKDIR /docker_api
COPY . /docker_api/
RUN pip install -r requirements.txt

# Run the app.  CMD is required to run on Heroku
# $PORT is set by Heroku
CMD python manage.py runserver 0.0.0.0:$PORT
```

比較要注意的就是最後一行，首先，這邊我們就先簡單的使用 `python manage.py runserver`，

反正現在已經在 Docker 裡面了，如果你未來想要加上 gunicorn 或是 uwsgi，請再自行修改:smirk:

另外一點是 `$PORT`，這個是由 Heroku 設定的。

如果佈署到 Heroku 上，我們會使用 [Heroku Postgres](https://www.heroku.com/postgres)，

可以參考 [Pushing your containers to Heroku](https://devcenter.heroku.com/articles/local-development-with-docker-compose#pushing-your-containers-to-heroku) 這邊的說明，

```text
Use Heroku add-ons in production
    * For local development: use official Docker images, such as Postgres and Redis.
    * For staging and production: use Heroku add-ons, such as Heroku Postgres and Heroku Redis.
```

[settings](https://github.com/twtrubiks/Deploying_Django_To_Heroku_With_Docker/blob/master/django_rest_framework_tutorial/settings.py) 中設定 db 的部分如下，

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': os.environ.get('DATABASE_NAME'),
        'USER': os.environ.get('DATABASE_USER'),
        'PASSWORD': os.environ.get('DATABASE_PASSWORD'),
        'HOST': os.environ.get('DATABASE_HOST'),
        'PORT': os.environ.get('DATABASE_PORT'),
    }
}
```

到時候這些環境變數會設定到 Heroku 的後台。

接著先到 Heroku 上建立一個 app，建立完 app 之後，再去建立 [Heroku Postgres](https://www.heroku.com/postgres)，

可參考之前我的文章 [如何在 heroku 上使用 database](https://github.com/twtrubiks/Deploying-Flask-To-Heroku#%E5%A6%82%E4%BD%95%E5%9C%A8-heroku-%E4%B8%8A%E4%BD%BF%E7%94%A8-database)。

建立完之後，你會看到 db 的連線資訊，

![alt tag](https://i.imgur.com/TsgF88a.png)

這些連線資訊可以先設定到 Heroku 的後台，

![alt tag](https://i.imgur.com/iZRaqRH.png)

那接著因為我們希望先在本地端執行，但我們需要 Heroku 上的環境變數才可以執行，那該怎麼辦:question:

Heroku 有提供在 local 環境中設定 Heroku 上的環境變數的方法，使用方法就是建立一個 `.env` 檔案

( 可參考[setting-multiple-environment-variables](https://devcenter.heroku.com/articles/container-registry-and-runtime#setting-multiple-environment-variables) )，

然後將環境變數設定在裡面 ( win 用戶請在 terminal 中輸入 `touch .env` 建立檔案 )。

[.env](https://github.com/twtrubiks/Deploying_Django_To_Heroku_With_Docker/blob/master/.env) 內容大致如下，

```text
PORT=8000
DATABASE_NAME=<your heroku db name>
DATABASE_USER=<your heroku db user>
DATABASE_PASSWORD=<your heroku db password>
DATABASE_HOST=<your heroku db host>
DATABASE_PORT=5432
```

一般來說 `.env` 應該要被寫入 `.dockerignore` 裡面，但這邊方便大家測試，我就不寫進去了。

到這邊我們終於可以在本地端啟動 Docker 來測試了:satisfied:

( db 的部分直接使用 [Heroku Postgres](https://www.heroku.com/postgres)，所以我才將連線資訊都寫進 `.env` 裡面 )。

接下來就是 Docker 指令而已，首先，

先 build image ( 使用命令提示字元切換到 `Deploying_Django_To_Heroku_With_Docker` 底下 )，

```cmd
docker build --tag web_image .
```

![alt tag](https://i.imgur.com/VCZkTBl.png)

如果順利 build 成功，使用下列指令會看到 web_image 這個 image，

```cmd
docker images
```

![alt tag](https://i.imgur.com/x3rADSF.png)

最後就是把它啟動，

```cmd
docker run -p 8000:8000 --env-file .env web_image
```

`.env` 前面說明過了，主要是方便 local environment test，可參考 [Setting multiple environment variables](https://devcenter.heroku.com/articles/container-registry-and-runtime#setting-multiple-environment-variables)，

![alt tag](https://i.imgur.com/vAztk85.png)

接下來，我們進去 container 中執行 db migrate，

```cmd
docker ps
docker exec -it xxxxx bash
```

![alt tag](https://i.imgur.com/bbyYWEi.png)

然後再 db migrate 以及建立 superuser ( 這邊的 db 是使用 Heroku 上的 )，

```cmd
python manage.py makemigrations musics
python manage.py migrate
python manage.py createsuperuser
```

建立完之後，可以瀏覽 [http://127.0.0.1:8000/api](http://127.0.0.1:8000/api)，

如果正確無誤，應該會看到下圖，

![alt tag](https://i.imgur.com/Rqk4kPK.png)

## Deploying Django To Heroku With Docker

既然在本地端測試沒問題，那現在我們就可以佈署到 Heroku 了，步驟只有幾個，

首先，使用命令提示字元切換到 `Deploying_Django_To_Heroku_With_Docker` 底下，

接著使用以下指令，

Build the image and push to Container Registry

```cmd
heroku container:push web -a <your heroku app name>
```

先說明這個 web，這個 web 的意思其實就像之前說的 Procfile 一樣，像是 [Procfile](https://github.com/twtrubiks/Deploying-Flask-To-Heroku/blob/master/Procfile)。

![alt tag](https://i.imgur.com/QevIz3k.png)

從上圖可以發現，其實他就是抓目錄資料夾底下是否有 [Dockerfile](https://github.com/twtrubiks/Deploying_Django_To_Heroku_With_Docker/blob/master/Dockerfile) 的檔案，

另外，`sdadss` 是我建立的 Heroku app name ( 請使用你自己的 app name， 你建立的 name 不會和我一樣 )。

如果順利執行，接著就是 release the image to your app，

```cmd
heroku container:release web -a <your heroku app name>
```

![alt tag](https://i.imgur.com/wPvWWSf.png)

最後再執行以下指令，

```cmd
heroku open -a <your heroku app name>
```

會自動幫你開一個網頁，如果看到畫面，就代表你成功使用 Docker 佈署了:smiley:

(也可以順便玩玩新增資料是否正常，也就是測試資料庫，這邊不需要再 migrate 了，

因為我們在本機端測試時就已經 migrate 了 )


[範例連結](https://sdadss.herokuapp.com/api/)

測試帳號和密碼如下，

```text
account : test123
password : password123
```

## 後記

整體來說，這次帶大家嘗試了使用 Docker 佈署 Django 到 Heroku 上，步驟算是還好，就一些小地方要注意，

不過我個人是覺得有些缺點，像是沒辦法使用 docker-compose 的方式佈署，docker-compose 只支持本地端的測試 ，

( 可參考 [Local Development with Docker Compose](https://devcenter.heroku.com/articles/local-development-with-docker-compose) )，

希望有一天可以支援，不然現在如果你有很多的 container ( 也就是使用 docker-compose )，要佈署到 Heroku 上會很麻煩:tired_face:

總之，以上就是帶大家體驗如何使用 Docker 佈署到 Heroku 上:sunglasses:

## 執行環境

* Python 3.6.2
* Win 10

## Reference

* [Container Registry & Runtime (Docker Deploys)](https://devcenter.heroku.com/articles/container-registry-and-runtime)
* [Local Development with Docker Compose](https://devcenter.heroku.com/articles/local-development-with-docker-compose)

## Donation

文章都是我自己研究內化後原創，如果有幫助到您，也想鼓勵我的話，歡迎請我喝一杯咖啡:laughing:

![alt tag](https://i.imgur.com/LRct9xa.png)

[贊助者付款](https://payment.opay.tw/Broadcaster/Donate/9E47FDEF85ABE383A0F5FC6A218606F8)

## License

MIT license
