# New Relic始め方
↓に従えばよいはずだが、無視して下の説明に進もう。  
https://docs.newrelic.co.jp/docs/agents/python-agent/installation/standard-python-agent-install

## ISUCON9-qualify for Python

+ クライアントのインストール。
```
cd webapp/python
pip install newrelic
```

+ ポータルから`APM`の`Get started`クリック->Pythonを選択
+ アプリ名を設定し、`newrelic.ini`をダウンロード->`webapp/python`に配置
+ アプリケーションを起動
```
NEW_RELIC_CONFIG_FILE=newrelic.ini newrelic-admin run-program python app.py
```