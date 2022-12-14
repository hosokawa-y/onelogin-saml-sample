## Onelogin toolkit（Django）のサンプル
https://github.com/onelogin/python3-saml

### セットアップ
1. Djangoのインストール
```
pip install django==1.11.29
```

2. python3-samlのインストール
```
pip install python3-saml
```

3. ローカル起動できるか確認
```
python manage.py runserver
```

### sslserverの導入
ローカル起動時にhttpだとChrome側の警告で表示ができないため、httpsで起動できるように `sslserver`を導入する

参考：[数行でできる Django https有効化](https://qiita.com/Syoitu/items/6205774c6348bc61df90)

1. sslserverのインストール
```
pip install django-sslserver
```

2. settings.pyの編集

```python
INSTALLED_APPS = [
    ...
    'sslserver'
]
...
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
SECURE_SSL_REDIRECT = False
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
```

3. 証明書の作成
```bash
openssl genrsa -out foobar.key 2048
openssl req -new -key foobar.key -out foobar.csr
openssl x509 -req -days 365 -in foobar.csr -signkey foobar.key -out foobar.crt
```

4. サーバーの立ち上げ
```
python manage.py runsslserver --certificate /path/to/.crt --key /path/to/.key
```

5. ポートを変更していなければ `https://127.0.0.1:8000/` にアクセス
6. Chromeで `「この接続ではプライバシーが保護されません」` が出たら Chromeがアクティブな状態で `thisisunsafe` と入力
(入力するボックスなどはないので、そのページが表示されている状態で打ち込む)
7. しばらく待つと `https://127.0.0.1:8000/` が表示される。

### SPとIdPの登録

saml/settings.json

```json
{
    "strict": true,
    "debug": true,
    "sp": {
        "entityId": "https://{アプリのURL}/metadata/",
        "assertionConsumerService": {
            "url": "https://{アプリのURL}/?acs",
            "binding": "urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST"
        },
        "singleLogoutService": {
            "url": "https://{アプリのURL}/?sls",
            "binding": "urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect"
        },
        "NameIDFormat": "urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified",
        "x509cert": "",
        "privateKey": ""
    },
    "idp": {
        "entityId": "IdPのエンティティID",
        "singleSignOnService": {
            "url": "IdPのSSO URL",
            "binding": "urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect"
        },
        "singleLogoutService": {
            "url": "https://app.onelogin.com/trust/saml2/http-redirect/slo/<onelogin_connector_id>",
            "binding": "urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect"
        },
        "x509cert": "509証明書"
    }
}
```