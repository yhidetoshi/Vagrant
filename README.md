# Vagrantを使う

#### 準備(CentOS6を使う)
#### Vagrantのダウンロード/インストール
  - https://www.vagrantup.com/downloads.html
  
  
#### VirtualBoxのダウンロード/インストール  
  - https://www.virtualbox.org/
  
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


#### 1つのboxから複数台のVMを作成/IP設定など
```
Vagrant.configure(2) do |config|
  config.vm.define :chef_client1 do |chef_client1|
    chef_client1.vm.box = "Base-OS-Cent"
    config.vm.network "private_network", ip: "192.168.33.10"
  end
  
  config.vm.define :chef do |chef|
    chef.vm.box = "Base-OS-Cent"
    config.vm.network "private_network", ip: "192.168.33.11"
  end
end
```

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

  
  
  
  
