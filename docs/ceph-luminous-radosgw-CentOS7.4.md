

### 2.5. Cấu hình /etc/ceph/ceph.conf trên từng host
- Thêm các dòng cấu hình radosgw sau ở cuối file, lưu ý thay đổi tên host tương ứng
	```sh
	[client.rgw.gatewaygw]
	host = radosgw
	rgw socket path = /var/run/ceph/ceph.radosgw.gateway.fastcgi.sock
	log file = /var/log/radosgw/client.radosgw.gateway.log
	rgw dns name = radosgw1.longlq.com
	rgw print continue = false
	```

### 2.6. Chuyển ceph.conf từ host ceph1 sang host ceph radosgw
```
scp /etc/ceph/ceph.conf  root@radosgw:/etc/ceph/ceph.conf
```

### 2.7. Chuyển ragosgw keyring từ host ceph1 sang host ceph radosgw
```
 
```


RADOSGW HOST
cd /etc/yum.repos.d

vim ceph-luminous.repo

```
[c7-ceph]
name=Ceph Aliyun for Enterprise Linux 7 - $basearch
baseurl=https://download.ceph.com/rpm-luminous/el7/x86_64/
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7

[c7-ceph-deploy]
name=Ceph-deploy for Enterprise Linux 7 - $basearch
baseurl=https://download.ceph.com/rpm-luminous/el7/noarch/
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
```

vim epel-7.repo

```
[epel]
name=Extra Packages for Enterprise Linux 7 - $basearch
baseurl=http://mirrors.aliyun.com/epel/7/$basearch
        http://mirrors.aliyuncs.com/epel/7/$basearch
#mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-7&arch=$basearch
failovermethod=priority
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
 
[epel-debuginfo]
name=Extra Packages for Enterprise Linux 7 - $basearch - Debug
baseurl=http://mirrors.aliyun.com/epel/7/$basearch/debug
        http://mirrors.aliyuncs.com/epel/7/$basearch/debug
#mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-debug-7&arch=$basearch
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=0
 
[epel-source]
name=Extra Packages for Enterprise Linux 7 - $basearch - Source
baseurl=http://mirrors.aliyun.com/epel/7/SRPMS
        http://mirrors.aliyuncs.com/epel/7/SRPMS
#mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-source-7&arch=$basearch
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=0
```

yum update -y

yum install -y ceph-radosgw

tren ceph1
ceph-deploy install --rgw --release luminous radosgw

ceph-deploy rgw create radosgw
Sau buoc nay user client.rgw.radosgw se duoc tao ra, tu ban luminous day se la user su dung de giao tiep voi ceph, khai bao user nay vao file ceph.conf
```sh
	[client.rgw.gatewaygw]
	host = radosgw
	rgw socket path = /var/run/ceph/ceph.radosgw.gateway.fastcgi.sock
	log file = /var/log/radosgw/client.radosgw.gateway.log
	rgw dns name = radosgw1.longlq.com
	rgw print continue = false
	```


ps -ef|grep radosgw 
netstat -nplt|grep 7480

yum install bind* -y

### 3.7. Cấu hình Private DNS trên host RadosGW
 


 - Sửa file /etc/named.conf
	```
	# Thu muc mac dinh chua cac zone file
	directory "/var/cache/bind";
	# Cau hinh dich vụ DNS cua host radosgw (172.16.69.163) su dung port 53
	listen-on port 53 { 127.0.0.1;172.16.69.77; };
	# Cau hinh dai mang cho phep query DNS
    allow-query      { localhost;172.16.69.0/24; };

 - Sửa file /etc/named.conf
 	```
 	# Them zone cho domain "demo.com"
 	zone "demo.com" IN {
		type master;
		file "db.demo.com";
		allow-update { none; };
	};
 	```
 - Tạo zone file db.demo.com tại thư mục /var/named
 	```
 	@ 86400 IN SOA demo.com. root.demo.com. (
        20091028 ; serial yyyy-mm-dd 
        10800 ; refresh every 180 min 
        3600 ; retry every hour
        3600000 ; expire after 1 month + 
        86400 ); min ttl of 1 day
	@ 86400 IN NS demo.com.
	@ 86400 IN A 172.16.69.163
	* 86400 IN CNAME @
 	```
 - Khởi động lại dịch vụ bind
 	```
 	systemctl start named.service
 	```

 Test:
 named-checkzone demo.com /var/named/db.demo.com
 Ket qua
 zone demo.com/IN: loaded serial 20091028
 OK

nslookup longlq.demo.com

Ket qua
Server:         172.16.69.77
Address:        172.16.69.77#53

Owncloud

yum install -y s3cmd

s3cmd --configure

vim /root/.s3cfg
host_base = radosgw.demo.com:7480
host_bucket = %(bucket)s.radosgw.demo.com:7480

[root@radosgw system]# cat /etc/resolv.conf 
; generated by /usr/sbin/dhclient-script
search demo.com
nameserver 172.16.69.77

s3cmd ls
s3cmd mb s3://owncloud
s3cmd del --recursive s3://owncloud

ceph --admin-daemon /var/run/ceph/ceph-client.rgw.radosgw.asok config show | grep dns

http://blog.csdn.net/for_tech/article/details/51098251
http://blog.csdn.net/for_tech/article/details/77188121