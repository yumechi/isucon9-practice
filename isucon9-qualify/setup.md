# isucon9-qualify データベースをホストの mysql ではなく docker から作る

Docker で構築する際に文字コードとかで辛い思いしたのでメモ

> 注意として[前準備](https://github.com/isucon/isucon9-qualify#%E5%89%8D%E6%BA%96%E5%82%99)を終わらせておくこと。コレをやらないと initial.sql っていう初期データが作成されない。

[最速で日本語環境の MySQL Docker コンテナを建てる方法](https://qiita.com/muff1225/items/48e0753e7b745ec3ecbd)があるそうなので、こちらを参考に Dockerfile を作る。

### 日本語環境を持った Dockerfile を作成

```bash
cat > ./webapp/sql/Dockerfile-MySQL <<EOF
# Dockerfile_MySQL
FROM mysql:5.7

# Set debian default locale to ja_JP.UTF-8
RUN apt-get update && \
    apt-get install -y locales && \
    rm -rf /var/lib/apt/lists/* && \
    echo "ja_JP.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen ja_JP.UTF-8
ENV LC_ALL ja_JP.UTF-8

# Set MySQL character
RUN { \
    echo '[mysqld]'; \
    echo 'character-set-server=utf8mb4'; \
    echo 'collation-server=utf8mb4_general_ci'; \
    echo '[client]'; \
    echo 'default-character-set=utf8mb4'; \
} > /etc/mysql/conf.d/charset.cnf
EOF
```

### 立ち上げやすくするために docker-compose.yaml 作成

```bash
cat > ./webapp/sql/docker-compose.yaml << EOF
mysql:
  build: .
  dockerfile: Dockerfile-MySQL
  environment:
    MYSQL_ROOT_PASSWORD: root
    TZ: "Asia/Tokyo"
  ports:
    - 3306:3306
  volumes:
    - ./db:/var/lib/mysql
    - ./initsql:/docker-entrypoint-initdb.d
EOF
```

### docker-entrypoint-initdb.d に格納する sql をまとめる

```bash
mkdir -p ./webapp/sql/initsql && cp ./webapp/sql/*.sql ./webapp/sql/initsql/
```

### gitignore に db と initsql を追加する

```bash
cat >> ./webapp/sql/.gitignore << EOF

db
initsql
EOF
```

### db を起動する

```bash
cd ./webapp/sql && docker-compose up
```

注意として、docker-compose.yaml に`MYSQL_USER`や`MYSQL_DATABASE`を**指定しない**こと。これをしちゃうとユーザが存在してた場合に適切に権限が当たらなかったり、docker-entrypoint-initdb.d に格納された sql が実行されないとかが発生してしまう。
