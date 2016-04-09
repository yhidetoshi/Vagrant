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


#### **[cloud-config]**で設定できること
===
- Path `/usr/share/oem/`
- etcd/fleet/などCoreOSのコンポーネントの動作
- 自動アップデート可否/アップデート方法
- 起動するsystemdユニット
- coreユーザーのssh_authorized_keyファイルに追加するSSH公開鍵
- ホスト名
- 「core」以外のユーザー追加
- 指定したパスへのファイル追加

===

**[メモ]**
- NW関連を設定
`/etc/systemd/network`の配下に
- token取得
```
curl -w "\n" 'https://discovery.etcd.io/new?size=3'
```
- cloud-configファイルの反映する場合
```
$ sudo coreos-cloudinit --from-file /usr/share/oem/cloud-config.yml
```
- etcdクラスタの確認
```
$ systemctl status etcd
```

### [Vagrant + etcd でクラスタを作る(Vagrantfileによる自動構築) ]
```
$ git clone https://github.com/coreos/coreos-vagrant.git
$ mv config.rb.sample config.rb
$ user-data.sample user-data
```

- Vagrantfileを書き換える(12行目と19行目　*今回は3VM)
```
  1 # -*- mode: ruby -*-
  2 # # vi: set ft=ruby :
  3
  4 require 'fileutils'
  5
  6 Vagrant.require_version ">= 1.6.0"
  7
  8 CLOUD_CONFIG_PATH = File.join(File.dirname(__FILE__), "user-data")
  9 CONFIG = File.join(File.dirname(__FILE__), "config.rb")
 10
 11 # Defaults for config options defined in CONFIG
 12 $num_instances = 3
 13 $instance_name_prefix = "core"
 14 $update_channel = "alpha"
 15 $image_version = "current"
 16 $enable_serial_logging = false
 17 $share_home = false
 18 $vm_gui = false
 19 $vm_memory = 512
```

- config.rbを書き換える (2行目 *今回は3VM)
```
 1 # Size of the CoreOS cluster created by Vagrant
 2 $num_instances=3
```
- vagrantを起動
```
$vagrant up
-> 起動したら名前は[core-01,core-02,core-03]となる
```

- クラスタの確認

core@core-01 ~ `$ etcdctl cluster-health`
```
member f20806a25c32b7f is healthy: got healthy result from http://172.17.8.103:2379
member 7bb180c680460ed7 is healthy: got healthy result from http://172.17.8.101:2379
member f7ce9760ed075f1e is healthy: got healthy result from http://172.17.8.102:2379
cluster is healthy
```
-> これでcore-01,02,03でクラスタが組めていることが確認できた


### [Vagrant + etcd2 でクラスタを作る(Vagrantで箱を作り、cloud-configは手動) ]
Vagrant3台を用意(core-01,core02,core03)
-> Vagrantfileで作成
```
VAGRANTFILE_API_VERSION = "2"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.define :core1 do |core1|
    core1.vm.box = "coreos-alpha"
    core1.vm.hostname = "core-01"
    core1.vm.network "private_network", ip: "172.17.8.101"
  end

  config.vm.define :core2 do |core2|
    core2.vm.box = "coreos-alpha"
    core2.vm.hostname = "core-02"
    core2.vm.network "private_network", ip: "172.17.8.102"
  end

  config.vm.define :core3 do |core3|
    core3.vm.box = "coreos-alpha"
    core3.vm.hostname = "core-03"
    core3.vm.network "private_network", ip: "172.17.8.103"
  end

 end
```
|hostname    |IPアドレス   |
|:-----------|:------------|
|core-01|172.17.8.101|
|core-02|172.17.8.102|
|core-03|172.17.8.103|

- cloud-configのフラグ説明
(参考:http://qiita.com/coreos/items/2e4ca397031d368832c4) 引用させて頂きました.
```
- name: 各メンバを識別するための固有の任意名を付けます。ここで付けた名前がinitial-clusterフラグで使用されます。
例: name: "member001"

- listen-client-url: etcdの基本的な通信において、受け付けるクライアントからのトラフィックを指定します。
メンバのインターナルIPとループバックアドレスをURL形式で指定します。
0.0.0.0を指定することで、すべてのインターフェースにおける指定したportを開けることができるます。これを使ってインターナルIPとループバックアドレスの両方を指定することを省略することができます。
例: listen-client-url: "http://0.0.0.0:2379" 
または、listen-client-url: "http://192.168.0.11:2379,http://127.0.0.1:2379"

- advertise-client-urls: etcdの基本的な通信において、ほかのメンバに公開するURLを指定します。
メンバのインターナルIPを指定します。
例: advertise-client-urls: "http://192.168.0.11:2379"

- listen-peer-urls: クラスタの通信において、受け付けるURLを指定します。
メンバのインターナルIPを指定します。peerはクラスタに関する通信なのでポートは2380番です。
例: listen-peer-urls: "http://192.168.0.11:2380"

- initial-advertise-peer-urls: クラスタの通信において、ほかのメンバに公開するURLを指定します。
メンバのインターナルIPを指定します。
例: initial-advertise-peer-urls: "http://192.168.0.11:2380"

- initial-cluster: クラスタを組む各メンバのURLをメンバ名=URLの形式で指定します。ここで指定するURLは、initial-advertise-peer-urlsフラグで指定したURLとイコールとなります。
静的にクラスタを組む時にのみ必要となるフラグです。
例: initial-cluster: "member001=192.168.0.11:2380,member002=192.168.0.12,member003=192.168.0.13"

- initial-cluster-token: クラスタを識別するためのトークンです。1つのクラスタ内の各メンバに対して同じトークンを設定します。トークンは任意の文字列です。
静的にクラスタを組む時にのみ必要となるフラグです。
例: initial-cluster-token: "cluster001"

- initial-cluster-state: newとexistingを指定でき、newはクラスタを新規に作成する場合に指定します。existingは、既存のクラスタにメンバを追加する場合に使用します。
例: initial-cluster-state: "new"
```


- 3台のcloud-configは以下に配置

`https://github.com/yhidetoshi/Vagrant/tree/master/coreos-vagrant/cloud-config-cluster`

- cloud-configを書き換える
```
$ sudo vi /usr/share/oem/cloud-config.yml
```

- ymlファイルに問題がないか確認する
```
$ sudo coreos-cloudinit --from-file /usr/share/oem/cloud-config.yml
```

- 問題が無ければ再起動
```
$ sudo reboot
```
- クラスタの確認
```
$ etcdctl cluster-health

member 3c9db69d7349bc1d is healthy: got healthy result from http://172.17.8.103:2379
member 8fe60b3a8c0f2724 is healthy: got healthy result from http://172.17.8.101:2379
member eb0bc02e1f6ccc95 is healthy: got healthy result from http://172.17.8.102:2379
cluster is healthy
```
