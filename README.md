# laravel-admin-develop  
Vagrantにより、Debian9をHostOSとしたLaraDockコンテナ群を生成し、Laravel8.0、Laravel-Adminの空のプロジェクトを生成します。  
  
実行すると、php7 + Laravel8 + Laravel-Admin1.8 + nginx + postgresql-postgis がインストールされます。


## 使用方法
TortoiseGit等でWindowsの適当なディレクトリ上にGitクローンしたらコンソールより下記を実行します。  
※実行前にVagrantfileの「APP_NAME = "sample_app"」でアプリ名を適宜変更して下さい。  
※環境にもよりますが１時間位はゆうに掛かります。  
```
vagrant up && vagrant ssh
```
Laravel  
http://192.168.33.50  
Laravel-Admin  
http://192.168.33.50/admin  

## 必要なもの
* Vagrant  
https://www.vagrantup.com/downloads.html
* VirtualBox  
https://www.oracle.com/technetwork/server-storage/virtualbox/downloads/index.html?ssSourceSiteId=otnjp
* TortoiseGit、テキストエディタ等

## Laravel-AdminのCRUD画面作成チュートリアル
### マイグレーション作成
workspace コンテナに入る
  
	Host# cd /opt/app/Laradock
	Host# docker-compose exec workspace bash
	
マイグレーションの殻を作成

    # php artisan make:migration create_staffs_table --create=staffs
    
　⇒　database\migrations\2020_09_14_071502_create_staffs_table.php　が生成される

マイグレーションを編集し、スキーマを設定

```
	$table->increments('id');
	$table->string('name');
	$table->text('description');
	$table->string('picture');
	$table->string('email');
	$table->boolean('active');
	$table->integer('age');
	$table->timestamps();

```
マイグレーションの実行
	# php artisan migrate

モデルの殻を作成
	# php artisan make:model Staff

　⇒　app\Models\Staff.php　が生成される

モデルを編集し、protected $table = 'staffs';　を追加

コントローラの殻を作成

    # php artisan admin:make StaffController --model='App\Models\Staff'

　⇒　Admin/Controllers/StaffController.php　が生成される

コントローラを編集し、一覧画面とフォーム画面を構築する

* grid()メソッド内

	    $grid->id('ID')->sortable();
	    $grid->column('name');
	    $grid->picture('picture')->image();
	    $grid->column('email');
	    $grid->active()->value(function ($active) {
	        return $active ?
	            "<i class='fa fa-check' style='color:green'></i>" :
	            "<i class='fa fa-close' style='color:red'></i>";
	    });
	    $grid->column('age');
	    $grid->created_at();
	    $grid->updated_at();

* form()メソッド内

        $form->display('id', 'ID');
        $form->text('name');
        $form->textarea('description');
        $form->image('picture');
        $form->text('email');
        $form->switch('active');
        $form->slider('age')->options(['max' => 100, 'min' => 1, 'step' => 1, 'postfix' => 'years old']);
        $form->display('created_at', 'Created At');
        $form->display('updated_at', 'Updated At');

ルーティング（admin/app/Admin/routes.php）を設定

    $router->resource('staffs', StaffController::class);

Laravel-Admin用のコンフィグを設定

	config\filesystems.php
	    'disks' 内に下記を追加
	
	        'admin' => [
	            'driver' => 'local',
	            'root' => storage_path('app'),
	        ],
以下で画面を開きます  
	http://[host-ip-address]/admin/staffs
  
  
