# Vagrantを使う

#### 準備(CentOS6を使う)
1.Vagrantのインストール
  - https://www.vagrantup.com/downloads.html
  
  
2.VirtualBoxのインストール  
  - https://www.virtualbox.org/
  
3.Boxの追加

```
# vagrant box add centos65 https://github.com/2creatives/vagrant-centos/releases/download/v6.5.3/centos65-x86_64-20140116.box

==> box: Box file was not detected as metadata. Adding it directly...
==> box: Adding box 'centos65' (v0) for provider:
    box: Downloading: https://github.com/2creatives/vagrant-centos/releases/download/v6.5.3/centos65-x86_64-20140116.box
==> box: Box download is resuming from prior download progress
==> box: Successfully added box 'centos65' (v0) for 'virtualbox'!
```

4.仮想マシンを作成/起動　
```
# mkdir -p ~/Vagrant/CentOS65
# cd Vagrant/CentOS65
# vagrant up
```
5.仮想マシンの状態確認
```
# vagrant status                                                                                                
Current machine states:
default                   running (virtualbox)
```
