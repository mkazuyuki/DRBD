# DRBD dual primary + OCFS2 で 2ノード分散環境で 共有ディスクを使わずに クラスタファイルシステム を構成する方法

下記URLを参考。

https://m.igrs.jp/blog/2011/05/27/dual-primary-with-drbd-ocfs2/

Ubuntu 16.04 に apt で drbd-utils と ocfs2-tools を入れる。(なので、使用したのは drbd 9 ではなく 8)

DRBD の 設定ファイル、ocfs2の設定ファイル、OS、kernel の情報は以下の通り。

----------
# DRBD の設定

	clp@ubuntu16-1:~$ cat /etc/drbd.d/r0.res
	resource r0 {
	  protocol C;

	  startup {
	    wfc-timeout 30;
	    degr-wfc-timeout 10;
	    become-primary-on both;
	  }

	  syncer {
	    rate 1G;
	  }

	  disk {
	    on-io-error detach;
	  }

	  net {
	    allow-two-primaries;
	    after-sb-0pri discard-zero-changes;
	    after-sb-1pri discard-secondary;
	    after-sb-2pri disconnect;
	  }

	  on ubuntu16-1 {
	    device    /dev/drbd0;
	    disk      /dev/sdb;
	    address   192.168.137.101:7789;
	    meta-disk internal;
	  }

	  on ubuntu16-2 {
	    device    /dev/drbd0;
	    disk      /dev/sdb;
	    address   192.168.137.102:7789;
	    meta-disk internal;
	  }
	}
	clp@ubuntu16-1:~$


# ocfs2 の設定

	clp@ubuntu16-1:~$ cat /etc/ocfs2/cluster.conf
	cluster:
	  node_count = 2
	  name = ocfs2

	node:
	  ip_port = 7777
	  ip_address = 192.168.137.101
	  number = 0
	  name = ubuntu16-1
	  cluster = ocfs2

	node:
	  ip_port = 7777
	  ip_address = 192.168.137.102
	  number = 1
	  name = ubuntu16-2
	  cluster = ocfs2
	clp@ubuntu16-1:~$


# OS kernelのバージョン

	clp@ubuntu16-1:~$ cat /etc/os-release
	NAME="Ubuntu"
	VERSION="16.04.4 LTS (Xenial Xerus)"
	ID=ubuntu
	ID_LIKE=debian
	PRETTY_NAME="Ubuntu 16.04.4 LTS"
	VERSION_ID="16.04"
	HOME_URL="http://www.ubuntu.com/"
	SUPPORT_URL="http://help.ubuntu.com/"
	BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
	VERSION_CODENAME=xenial
	UBUNTU_CODENAME=xenial
	clp@ubuntu16-1:~$
	clp@ubuntu16-1:~$ uname -a
	Linux ubuntu16-1 4.4.0-116-generic #140-Ubuntu SMP Mon Feb 12 21:23:04 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
	clp@ubuntu16-1:~$
