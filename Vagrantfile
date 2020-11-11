# -*- mode: ruby -*-
# vi: set ft=ruby :

# ---------------------------------------------------------
# config
# ---------------------------------------------------------
VM_BOX = "generic/debian9"
GUEST_IP = "192.168.33.60"
HOST_APP_DIR = "./app"
GUEST_APP_DIR = "/opt/app"
GUEST_MEM = 4096 # 単位：MB
GUEST_CPU = 4
APP_NAME = "kirokuman"
# ---------------------------------------------------------

GUEST_APP_DIR2 = GUEST_APP_DIR.gsub(/\//,'\\/')

Vagrant.configure(2) do |config|
  config.ssh.insert_key = false
  config.vm.box = VM_BOX
  config.vm.network "private_network", ip: GUEST_IP
  config.vm.network "forwarded_port", guest: 80, host: ENV['HOST_WEB_PORT'] if port = ENV['HOST_WEB_PORT']

  config.vm.provider "virtualbox" do |vb|
    vb.memory = GUEST_MEM
    vb.cpus = GUEST_CPU
    vb.customize ["modifyvm", :id, "--ioapic", "on"]

    # ホストがMac、LinuxでNFSマウントを希望する場合、下記を有効化する(* 未確認)
    #vb.check_guest_additions = false
    #vb.functional_vboxsf     = false
  end

  # vbguestの自動アップデートを無効にする
  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end

  # ディレクトリ共有
  config.vm.synced_folder HOST_APP_DIR, GUEST_APP_DIR, mount_options: ['dmode=777', 'fmode=777']
  # ディレクトリ共有　ホストがMac、LinuxでNFSマウントを希望する場合
  #config.vm.synced_folder HOST_APP_DIR, GUEST_APP_DIR, type: "nfs", mount_options: ['dmode=777', 'fmode=777']


  #初回起動時のみ実行（この中は\は\でエスケープすること）
  config.vm.provision :shell, :inline => <<-EOT
    sudo su -
	apt update
    # Install Docker
		# 参考：https://www.codeflow.site/ja/article/how-to-install-and-use-docker-on-debian-10
	apt install apt-transport-https ca-certificates curl gnupg2 software-properties-common -y
	curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
	add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
	apt update
	apt-cache policy docker-ce
	apt install docker-ce -y

    # Install Docker-compose
    wget -qL https://github.com/docker/compose/releases/download/1.27.0/docker-compose-`uname -s`-`uname -m`
    chmod +x docker-compose-`uname -s`-`uname -m`
    mv docker-compose-`uname -s`-`uname -m` /bin/docker-compose
    chown root:root /bin/docker-compose

    # JP LocaleSetting
		# 一旦スキップ

    # StartUp Shell Install
		# 一旦スキップ

    # Laradock Install
    cd #{GUEST_APP_DIR}
    git config --global http.sslVerify false
    git clone https://github.com/Laradock/laradock.git
    cd laradock
    cp env-example .env

    #データパスを外だしに変更
    sed -i -e "s/DATA_PATH_HOST=~\\/\\.laradock\\/data/DATA_PATH_HOST=\\/opt\\/app\\/data/" .env

    #mysql5.7に変更
    sed -i -e "s/MYSQL_VERSION=latest/MYSQL_VERSION=5.7/" .env

    # Image & Container build 超絶長い1～2時間位
    docker-compose up -d apache2 mysql
    echo "★★★コンテナビルド完了"

    # Laravel Install
    docker-compose exec -T workspace sh -c "composer create-project --prefer-dist laravel/laravel #{APP_NAME}"
    echo "★★★Laravelインスコ完了"

    sed -i -e "s/APP_CODE_PATH_HOST=\\.\\.\\//APP_CODE_PATH_HOST=#{GUEST_APP_DIR2}\\/#{APP_NAME}/" .env


    # アプリ側に移動
    cd ../#{APP_NAME}
    # ストレージのパーミッションを設定
    chmod 777 -R storage
    # PostgreSQLの接続設定（デフォはMySQLの為MySQLの設定は削除）
    sed -i -e "/^DB_/d" .env
    cat <<EOF >> .env
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=default
DB_USERNAME=default
DB_PASSWORD=secret
EOF

    # laradock側に移動
    cd ../laradock

    # Recreate Container（これ重要）
    docker-compose up -d apache2 mysql

    # laravel-Admin Install after pg migrating & seeding
    docker-compose exec -T workspace sh -c "composer require encore/laravel-admin:2.0.0-beta1"
    docker-compose exec -T workspace sh -c 'php artisan vendor:publish --provider="Encore\\Admin\\AdminServiceProvider"'
    docker-compose exec -T workspace sh -c "composer dump-autoload"
    docker-compose exec -T workspace sh -c "php artisan migrate:refresh --seed"
    docker-compose exec -T workspace sh -c "php artisan admin:install"

    sudo gpasswd -a vagrant docker

    echo ""
    echo "Laravel開発環境の構築が完了しました！"

  EOT


  #vagrant reload時用
  config.vm.provision :shell, run: "always", :inline => <<-EOT
    sudo su -
    cd #{GUEST_APP_DIR}/laradock
    pwd
    docker-compose up --no-recreate -d apache2 mysql
    docker-compose ps
	echo ""
	echo "Laravel開発環境を起動しました！"
	echo "Laravel：　http://#{GUEST_IP}"
	echo "Laravel-Admin：　http://#{GUEST_IP}/admin"
	echo "ログインユーザ / パスワード：　admin / admin(default)"
  EOT

end



