# Vagrantを使う

![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Vagrant/vagrant-icon.png)

#### Vagrantのダウンロード/インストール
  - https://www.vagrantup.com/downloads.html
  
  
#### VirtualBoxのダウンロード/インストール  
  - https://www.virtualbox.org/


#### 準備(CentOS6を使う)

![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Vagrant/centos-icon.png)

#### Boxの追加

```
# vagrant box add centos65 https://github.com/2creatives/vagrant-centos/releases/download/v6.5.3/centos65-x86_64-20140116.box

==> box: Box file was not detected as metadata. Adding it directly...
==> box: Adding box 'centos65' (v0) for provider:
    box: Downloading: https://github.com/2creatives/vagrant-centos/releases/download/v6.5.3/centos65-x86_64-20140116.box
==> box: Box download is resuming from prior download progress
==> box: Successfully added box 'centos65' (v0) for 'virtualbox'!
```

#### 仮想マシンを作成/起動　
```
# mkdir -p ~/Vagrant/CentOS65
# cd Vagrant/CentOS65
# vagrant init centos65
# vagrant up
```
#### 仮想マシンの状態確認
```
# vagrant status                                                                                                
Current machine states:
default                   running (virtualbox)
```

#### box名を変更する
- $HOME/.vagrant.d/boxesにインストールされているので`$ mv`で変更


#### 1つのboxから複数台のVMを作成/IP設定など(マルチ環境)
```
VAGRANTFILE_API_VERSION = "2"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.define :chef_client1 do |chef_client1|
    chef_client1.vm.box = "Base-OS-Cent"
    chef_client1.vm.hostname = "chef-client1"
    chef_client1.vm.network "private_network", ip: "192.168.33.10"
  end


  config.vm.define :chef_client2 do |chef_client2|
    chef_client2.vm.box = "Base-OS-Cent"
    chef_client2.vm.hostname = "chef-client2"
    chef_client2.vm.network "private_network", ip: "192.168.33.11"
  end

 config.omnibus.chef_version = :latest
  config.vm.provision :chef_solo do |chef|
    chef.cookbooks_path = "./cookbooks"

    chef.run_list = %w[
	recipe[httpd]
    ]
  end
 end
```
#### 作ったVMにsshする
- (not Multi環境) -> `$ vagrant ssh`
- (Multi環境)     -> `$ vagrant ssh <vm_name> | <ip_address>`


#### vagrantコマンドメモ　
====
|コマンド    |機能         |
|:-----------|:------------|
|$ vagrant up|仮想サーバーを起動|
|$ vagrant status|仮想サーバーの状態を確認|
|$ vagrant ssh|仮想サーバーにログイン|
|$ vagrant halt|仮想サーバーのシャットダウン|
|$ vagrant reload|仮想サーバーの再起動|
|$ vagrant suspend|仮想サーバーの一時停止|
|$ vagrant resume|仮想サーバーの再開|
|$ vagrant destroy|仮想サーバーを削除|

  
  
### CoreOSを使う

![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Vagrant/coreos-web.png)
![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Vagrant/coreos-icon.png)


CoreOSとは：コンテナを稼動させることに特化したLinuxディストリビューション。そのためかなり軽量。

**CoreOSの基盤**

- Docker
- etcd
- fleet
- Docker

CoreOS ではサービスやアプリケーションは全て Docker コンテナ内で動かす
コンテナ内で動かすことでプロセスの隔離、安全なリソース共有、アプリケーションのポータビリティを確保している

- **etcd**
 
複数の CoreOS ノードでクラスタを構成する際に利用する為の KVS 機能を提供する
KVS を利用して各種ノード間で設定を共有する

- **fleet**

CoreOS の各ノードで稼働させるアプリケーション、サービスコンテナのスケジューリングとコンテナの管理を行う


- **VagrantでCoreOSを使う**
```
$ git clone https://github.com/coreos/coreos-vagrant/
$ cd coreos-vagrant
$ vagrant up
```
- OSバージョンの確認
$ cat /etc/os-release


- /usr/share/oem/**cloud-config**で設定できること

- etcd/fleet/などCoreOSのコンポーネントの動作
- 自動アップデート可否/アップデート方法
- 起動するsystemdユニット
- coreユーザーのssh_authorized_keyファイルに追加するSSH公開鍵
- ホスト名
- 「core」以外のユーザー追加
- 指定したパスへのファイル追加

- NW関連を設定
`/etc/systemd/network`の配下に

- cloud-configファイルの反映する場合
```
$ sudo coreos-cloudinit --from-file /usr/share/oen/cloud-config
```


