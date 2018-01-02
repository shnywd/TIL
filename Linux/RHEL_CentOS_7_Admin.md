# RHEL/CentOS 7

### 特徴

* 64bitのみ
* systemd, firewalld を採用
* ファイルシステムのデフォルトにXFS を採用
  * 大容量ファイル
* GNOME クラシックと GNOME Shell が選択可能
  * GNOME クラシック : 従来のメニューバー、ウィンドウを使用するGUI
  * GNOME Shell : タブレットのようなアイコン表示のGUI
* 時刻同期が ntpd → chrony に変更

### ネットワーク

* ネットワークインタフェースの命名規則

eno1
ens1
enp2s0
enx78e7d1ea46da
wi1

ens32

*  ネットワークの確認

ip, ss, ping

```
# IPアドレスの確認
ip addr show

# ルーティングテーブルの確認
ip rout show

# ARPテーブルの確認
ip neighbor show

# セッションを確認する
## TCPポートで通信を行っているすべての通信状態
ss -nat
## TCPポートでリッスンしている全てのポート
ss -nlt
## TCPポートで通信を行っているすべての通信状態
ss -nau
## TCPポートでリッスンしている全てのポート
ss -nlu

# ネットワークの疎通を確認する
ping <IPアドレス or FQDN>

ping -n <回数> <IPアドレス or FQDN>
```

### ソフトウェアのインストール - yum

* RHELをカスタマーポータルに登録する

```
# サブスクリプションを登録する
subscription-manager register
# サブスクリプションをアタッチする
subscription-manager attach --auto
# アタッチされているサブスクリプションを確認する
subscription-manager list
```

* プロキシの設定

```
echo 'proxy http://<proxy_name>:<proxy_port>/' > /etc/yum.conf.d/proxy.conf
```

* パッケージの管理

※ 初回のみ公開鍵をインストールする

```
# パッケージを最新に更新する
yum update

# ソフトウェアのインストール
yum install <パッケージ名>

# 使用可能なパッケージを検索する
yum search <キーワード>

# パッケージ内のファイル名一覧を表示する

# ファイルがどのパッケージに属するか調べる
```


### サービスの管理 ###

```
# サービスを起動する
systemctl start httpd.service
systemctl stop httpd.service
systemctl status httpd.service

# OS起動時にサービスが自動起動するように設定する
systemctl disable httpd.service
systemctl enable httpd.service

# 起動時の動作モードを変更する
systemctl set-default multi-user.target
# 起動後に動作モードを変更する
systemctl isolate multi-user.target
```

| ランレベル | target |
|----------|--------|
| 0        | poweroff.target |
| 1        | rescue.target |
| 3        | multi-user.target |
| 5        | graphical.target |
| 6        | reboot.target |
| emergency | emergency.target |

### ユーザーを管理する

```
## [root]
# ユーザーを追加する ( -m ホームディレクトリも合わせて作成)
useradd -g users <アカウント名>
useradd -u <UID> -g <グループ名> -G <グループ2,...> -m <アカウント名>
# パスワードを変更する
passwd <アカウント名>
# グループを追加する
groupadd <グループ名>
groupadd -g <GID> <グループ名>
# ユーザーをグループに加入させる ( -a : 現在の設定に追加)
usermod -a -G <グループ名> <アカウント名>
# ユーザーを削除する ( -r : ホームディレクトリも合わせて削除)
userdel -r <ユーザー名>
```

### firewalld でファイアウォールを設定する

firewalld は目的のサービスに関連するポートを開放するという考え方で設定を行う。
実際のフィルタリングの処理に、Linux のパケットフィルタリングの仕組み "Netfilter" を使用している点は iptables と同じ。

* サービスの許可と禁止
```
# [root]
# ゾーンにサービスを追加する
firewall-cmd --add-service=http --zone=public
# 再起動後にも追加した設定を有効にする
firewall-cmd --remove-service=http --zone=public --permanent
# ゾーンからサービスを外す
firewall-cmd --add-service=http --zone=public
# 再起動後も外した設定を有効にする
firewall-cmd --remove-service=http --zone=public --permanent
```

許可されたサービスの一覧
```
firewall-cmd --list-services --zone=public
```

インタフェースのゾーンの変更
```
# ゾーンを変更する
firewall-cmd --change-interface=eno2 --zone=internal
# 再起動後にも変更を有効にする
firewall-cmd --change-interface=eno2 --zone=internal --permanent
```

### OpenSSHの設定

sshd の設定を編集する

/etc/ssh/sshd_config を編集する

```
## パスワード認証を禁止にする [YES->no]
PasswordAuthentication no
## ポートを変更する [22 -> 任意]
Port 22
## プロトコルを変更する [1,2 -> 2 のみ]
Protocol 2
## リモートからのrootログインを許可しない [no -> no]
PermitRootLogin no
```

設定を有効にする
```
## [root]
systemctl restart sshd.service
```

### ディスクの管理

GPT形式の場合はpartedコマンドかgdiskコマンドを使う

LVMの場合

1. partedを起動する
2. GPT形式を選ぶ
3. パーティションを作る
4. LVMフラグを有効にする
5. パーティションテーブルを表示する
6. partedを終了する
7. 物理ボリュームを初期化する
8. ボリュームグループを表示する
9. ボリュームグループに物理ボリュームを追加する
10. ボリュームグループを表示する
11. 論理ボリュームのサイズを拡張する
12. ファイルシステムのサイズを拡張する

```
## 1. [root]
parted

## 2-6. command (parted)
mklabel gpt
mkpart
# name, type=ext2, begin=1, end=100GB
set 1 lvm on
print
quit

# 7-12. [root]
pvcreated /dev/sdb1
vgs
vgextend rhel /dev/sdb1
vgs
lvresize -L +100G /dev/rhel/root
xfs_growfs /
```

* 起動時の定期的なファイルシステムチェックをOFFにする

再起動したときに予期しない fsck が実行されて、復旧にかかる時間が長くなるのを防ぐ

```
# 時間
tune2fs -i 0 /dev/sda1
# 回数
tune2fs -c 0 /dev/sda1

# 確認
tune2fs -l /dev/sda1 | egrep -i '(interval|count)'
```

### ブートローダーの設定

GRUB2
* BIOSブート、UEFIブートの両方をサポート


|            | GRUB (RHEL 6) | GRUB2 (RHEL 7) |
|-------------|-----------|----------|
| 設定ファイル | /boot/grub/grub.conf |/boot/grub2/grub.cfg |
|            | /boot/efi/EFI/redhat/grub.conf | /boot/efi/EFI/redhat/grub.cfg |
|            |          | /etc/default/grub |
|            |          |/etc/grub.d/* |
| GRUBモジュール | /boot/grub/*_stage1_5 | /boot/grub2/i386-pc/*.mod|
|     |   | /boot/grub2/x86_64-efi/*.mod |
| ブートローダーのインストール | grub-install | grub2-install |
| 設定ファイル生成 | - | grub2-mkconfig |
| パスワード生成 | grub-crypt | grub2-mkpasswd-pdkdf2 |
| デフォルトエントリー指定 | - | grub2-set-default |。

RHEL6までは通常の手順だった /boot/grub.conf や /boot/efi/EFI/redhat/grub.conf を直接編集するのは禁止になっている。
/etc/default/grub や /etc/grub.d/ ディレクトリの設定ファイルを編集してから、grub2-mkconfig コマンドを実行し設定を書き出す。

GRUB2の設定ファイル
* /boot/grub2/grub.cfg
* /boot/efi/EFI/redhat/grub.cfg
* /etc/default/grub
  * 00_header
  * 10_linux
  * 20_linux_xen
  * 20_ppc_terminfo
  * 30_os-prober
  * 40_custom
  * 41_custom

```
# BIOSブートの場合
grub2-mkconfig -o /boot/grub2/grub.cfg
# UEFIブートの場合
grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg
```

GRUB2のエントリーを確認する
```
grub2-editenv list
```

システム起動時に自動的に選ばれるデフォルトエントリーを切り替えたい場合
```
# デフォルトエントリーを変更する
grub2-set-default 2
```

GRUB2のパラメーターやカーネルオプションを変更する

/etc/default/grubの設定ファイルを変更する
* GRUB_TIMEOUT : GRUBのタイムアウト時間 (秒)
* GRUB_CMDLINE_LINUX :  カーネルオプション
```
GRUB_TIMEOUT=5
GRUB_CMDLINE_ILNUX="rd lvm.lv=vg_root/lv_root crashkernel=auto vconsole.font=latarcyrheb-sun16 rd.lvm.lv=vg_root/lv_swap vconsole.keymap=us rhgb quiet"
```

他のディストリビューションを起動する設定
```
menuentry "Other Linux" --class gnu-linux --class os {
  insmod gzio
  insmod part_msdos
  insmod ext2
  set root='hd0.msdos5'
  linux16 /boot/vmlinux-custom ro
  initrd16 /boot/initramfs-custom.img
}
```

* レスキューモード sysemd.unit=rescue.target
  * シングルユーザーモード runlevel=0 に相当するモード
* 緊急モード sysemd.unit=emergency.target
  * レスキューモードよりさらに深刻で、ルートファイルシステムすらマウントができない場合

カーネルオプションのエントリーの最後に移動し systemd.unit　オプションを追加する

```
# 通常のモードに戻す場合
systemctl default
# 再起動する場合
systemctl reboot
```

### バックアップ

インストールされているRPMのバックアップ

```
rpm -qa > rpm-qa.txt
rpm -qa --queryformat=''%{NAME}.%{ARCH}¥n' > rpm-qa-qf.txt
```
インストールされているRPMのリストア
```
subscription-manager register --auto-attach
while read RPM; do yum install -y %{RPM}; done < rpm-qa-qf.txt
```

ファイルのバックアップ

* --selinux --acls --xattrs
  <br />
  SELinuxのコンテキストもバックアップする

```
tar -C / --selinux --acls --xattrs -xcvf /opt/etc.tar.gz /etc
```

ファイルのリストア
```
tar -C / -zxvf /opt/etc.tar.gz
```


### パフォーマンス設定

* カーネルパラメータ
  * 物理 or 仮想
* ユーザーパラメータ
  * 開けるファイルの設定

  tuned
  用意されているプロファイル
  * throughput-performance
  * virtual-guest
  * balances
