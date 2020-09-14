# laravel-admin-develop  
Vagrantにより、LaraDockコンテナ群を生成し、Laravel8.0、Laravel-Adminのプロジェクトを生成します。  
  
実行すると、php7 + Laravel8 + Laravel-Admin1.8 + nginx + postgresql-postgis がインストールされます。


## 使用方法
TortoiseGit等でWindowsの適当なディレクトリ上にGitクローンしたらコンソールより下記を実行します。  
※実行前にVagrantfileの「APP_NAME = "sample_app"」でアプリ名を適宜変更して下さい。  
※環境にもよりますが１時間位はゆうに掛かります。  
```
vagrant up && vagrant ssh
```

## 必要なもの
* Vagrant  
https://www.vagrantup.com/downloads.html
* VirtualBox  
https://www.oracle.com/technetwork/server-storage/virtualbox/downloads/index.html?ssSourceSiteId=otnjp
* TortoiseGit、テキストエディタ等
