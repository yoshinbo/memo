#Bang開発関連コメントメモ
## migrationファイル作るとき
```
bundle exec rails g migration init
```

## 新しくモデル作るとき
``
# 開発中はdb/migrate/20150406041827_init.rbを編集するのでmigration作成は不要
bundle exec rails d model user --skip-migration
# モデルにコメントつけるgit commit hookに追加
bundle exec annotate
```

## DB作り直し
```
bundle exec rake db:rollback
bundle exec rake db:migrate —trace
```

## テスト準備
```
bundle exec rake db:test:prepare
bundle exec rspec
```

## symmetric-encryption用の鍵作成
```
bundle exec rails generate symmetric_encryption:config
```

# Vagrant
## 日本語対応
```
sudo localedef -f UTF-8 -i ja_JP ja_JP
```

## git 対応
```
git config --global color.ui true
git config --global user.name yoshinbo
git config --global user.email yoshinbo@gmail.com
git remote set-url origin https://yoshinbo@github.com/shiruco/bang-server.git
```

## vagrant環境でsubdomainを使った場合
mac側の/etc/hostsにsubdomain付きのdomainを追加
```
127.0.0.1 api.localhost.local
```

# PostMan
Headerに下記を指定
```
Content-Type => application/json
```
RequestParameterに下記を指定
```
{
  "user": {
    "facebook_id": "100",
    "name": "yoshikazu oda",
    "birthday": "1983/10/04",
    "gender": "male"
  }
}
```
