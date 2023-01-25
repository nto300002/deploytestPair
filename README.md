# Docker_Django_uWSGI_Sample

Docker + Django + Nginx + uWSGI + MySQL

##

## app_dir ディレクトリ内にプロジェクトを生成

#

# 開発環境設定

1. docker-compose.yml と同じ階層に開発環境「.env」ファイルと本番環境「.prod.env」ファイルを作成し記述

2. プロジェクトを新規作成

```
# docker compose -f <指定するdocker-composeのファイル> run app django-admin startproject <プロジェクト名> <プロジェクトを作成するディレクトリ>
docker compose -f docker-compose.yml run app django-admin startproject pairleary_project .
```

3. settings.py を編集

```
# osのモジュールをインポート
import os

# SECRET_KEYを.envから取得
SECRET_KEY = os.environ.get("SECRET_KEY")

# DEBUGを.envから取得
# envファイルにTrue、Falseと書くとDjangoがString型と認識してしまいます
# os.environ.get("DEBUG") == "True"を満たすとboolean型のTrueになり、
# env内のDEBUGがTrue以外ならFalseになります
DEBUG = os.environ.get("DEBUG") == "True"

# ALLOWED_HOSTSを.envから取得
ALLOWED_HOSTS = os.environ.get("DJANGO_ALLOWED_HOSTS").split(" ")

# MySQLのパラメータを.envから取得
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.mysql",
        # コンテナ内の環境変数をDATABASESのパラメータに反映
        "NAME": os.environ.get("MYSQL_DATABASE"),
        "USER": os.environ.get("MYSQL_USER"),
        "PASSWORD": os.environ.get("MYSQL_PASSWORD"),
        "HOST": os.environ.get("MYSQL_HOST"),
        "PORT": os.environ.get("MYSQL_PORT"),
    }
}

# 言語を日本語に設定
LANGUAGE_CODE = 'ja'
# タイムゾーンをAsia/Tokyoに設定
TIME_ZONE = 'Asia/Tokyo'

# STATIC_ROOTを設定
# Djangoの管理者画面にHTML、CSS、Javascriptが適用されます
STATIC_ROOT = "/static/"
STATIC_URL = "/static/"


# [・・・]

TEMPLATES = [
    {
     ....
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
     ....
]

```

4. 下記コマンドを実行する

```
docker compose -f docker-compose.yml up -d
```

- ブラウザで確認
  http://localhost:8000

5. マイグレートして、スーパーユーザーを作成する

```
./.migration.sh
```

- ブラウザで確認
  http://localhost:8000/admin

6. 一旦 Docker をリセットする

```
./.docker_clear.sh
```

7. Docker を起動

```
docker compose -f docker-compose.yml up -d --build
```

8. app_dir 内で Django アプリを開発して行く

# 本番環境設定

9. docker-compose.prod.yml の編集

```
uwsgi --socket :8000 --module [アプリ名].wsgi --py-autoreload 1 --logto /tmp/mylog.log
```

10. [wsgi.ini]作成編(記述途中)

- /<プロジェクト名>/uwsgi.ini を作成する

```
[uwsgi]
chdir            = /home/www-user/<プロジェクト名>
module           = <プロジェクト名>.wsgi:application
pidfile          = /run/<プロジェクト名>/<プロジェクト名>.pid
socket           = /run/<プロジェクト名>/<プロジェクト名>.sock
home             = /home/www-user/<プロジェクト名>/.venv
daemonize        = /home/www-user/<プロジェクト名>/<プロジェクト名>.log
uid              = www-user
gid              = www-users

master           = true
processes        = 5
harakiri         = 30
max-requests     = 5000
vacuum           = true

disable-logging  = true
log-4xx          = false
log-5xx          = true

env DJANGO_SETTINGS_MODULE = <プロジェクト名>.settings
```

11. 本番環境で立ち上げ

```
docker compose -f docker-compose.prod.yml up -d --build
```

- ブラウザで確認
  http://localhost

12. Docker の停止・削除

```
docker stop nginx

./.docker_clear.sh

```
