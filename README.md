# CentOS7 nginx django uwsgi
# 1. Install & Set python
Go to https://www.python.org/ftp/python/ and choose the version you need
```
$ wget https://www.python.org/ftp/python/3.6.12/Python-3.6.12.tgz
```
Unzip this tgz
```
$ tar -zxvf Python-3.6.12.tgz
$ cd Python-3.6.12/
```
If you want to change the command from 'python3.6' to 'python'	
Please refer to the code below	
```
$ which python3.6
$ vi /root/.bashrc
$ alias python = "/usr/local/bin/python3.6"
$ source /root/.bashrc
$ python -V
```
FYI, it is not recommended change the python symbolic link like:
```
$ rm -i /usr/bin/python
$ ln -s /usr/bin/python3.6 /usr/bin/python
```
It can cause errors as conflict between installed python and python3 you install	

```
$ yum install zlib-devel -y
$ yum install openssl openssl-devel -y

$ ./configure --prefix=/usr/local --enable-shared LDFLAGS="-Wl,-rpath /usr/local/lib"
$ make && make altinstall
>Installing collected packages: setuptools, pip
>Successfully installed pip-18.1 setuptools-40.6.2


$ curl -k -O https://bootstrap.pypa.io/get-pip.py
$ python3.6 get-pip.py

$ pip3 install Django
>Successfully installed Django-3.1.1 asgiref-3.2.10 pytz-2020.1 sqlparse-0.3.1
```

# 2. Install & Set MariaDB

```
$ curl -sS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | sudo bash
$ cd /etc/yum.repo.d/
$ ls
> mariadb.repo
$ vi mariadb.repo
# The mariadb version is 10.5 which is stable for centos7 (this is not fit centos6 so if you use centos6 you have to fix this version to 10.3 or lower)
```


Note that if you change an existing repository configuration, then you need you execute the following:	
```
$ sudo yum clean all
```

```
$ yum search MariaDB
$ yum install -y mariadb mariadb-server mariadb-devel
>Installed:
  MariaDB-client.x86_64 0:10.5.5-1.el7.centos    MariaDB-compat.x86_64 0:10.5.5-1.el7.centos    MariaDB-devel.x86_
64 0:10.5.5-1.el7.centos    MariaDB-server.x86_64 0:10.5.5-1.el7.centos

Dependency Installed:
  MariaDB-common.x86_64 0:10.5.5-1.el7.centos       boost-program-options.x86_64 0:1.53.0-28.el7      galera-4.x86
_64 0:26.4.5-1.el7.centos      pcre2.x86_64 0:10.23-2.el7
  perl-Compress-Raw-Bzip2.x86_64 0:2.061-3.el7      perl-Compress-Raw-Zlib.x86_64 1:2.061-4.el7       perl-DBI.x86_64 0:1.627-4.el7              perl-IO-Compress.noarch 0:2.061-2.el7
  perl-Net-Daemon.noarch 0:0.48-5.el7               perl-PlRPC.noarch 0:0.2020-14.el7                 socat.x86_64 0:1.7.3.2-2.el7

Replaced:
  mariadb-libs.x86_64 1:5.5.52-1.el7

$ systemctl enable mariadb.service
>Created symlink from /etc/systemd/system/multi-user.target.wants/mariadb.service to /usr/lib/systemd/system/mariadb.service.

$ systemctl start mariadb.service
$ mysql_secure_installation
> Enter current password for root (enter for none):
> Switch to unix_socket authentication [Y/n] Y
```

You can use the `unix_socket` method and the `mysql_native_password` method as it is.
That is, you can access both with `sudo mysql` and `mysql -u root -p`.

```
> Change the root password? [Y/n] Y
> New password:
> Re-enter new password:
> Password updated successfully!
>Remove anonymous users? [Y/n] Y
> ... Success!
> Disallow root login remotely? [Y/n] Y
> Remove test database and access to it? [Y/n] Y
> Reload privilege tables now? [Y/n] Y
```

```
$ yum search python3 | grep devel
> python3-devel.i686 : Libraries and header files needed for Python development
> python3-devel.x86_64 : Libraries and header files needed for Python development
> python3-idle.i686 : A basic graphical development environment for Python
> python3-idle.x86_64 : A basic graphical development environment for Python

$ yum install python3-devel.x86_64
# I found that the problem is that mysqlclient requires mysql-devel packages, which is different from mariadb-devel. Don't install mariadb-devel!
# So you need to:
# Remove MariaDB-devel which is installed when we install MariaDB-server
$ yum list installed | grep mariadb
$ sudo yum erase MariaDB-devel.x84_64

# Add MySQL repository in yum
# Go to https://dev.mysql.com/downloads/repo/yum/ and select the RPM file for your CentOS 
# (for me, I choose "Red Hat Enterprise Linux 7 / Oracle Linux 7 (Architecture Independent), RPM Package"
# Go to you terminal and type:
$ wget   https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
$ sudo rpm -Uvh mysql80-community-release-el7-3.noarch.rpm
$ sudo yum install mysql-community-devel.x86_64
$ sudo pip3 install mysqlclient
```

```
$ mysql -u root -p
> Enter password: 
> create database db_name;
> grant all privileges on db_name.* to user_name@localhost identified by 'password';
> grant all privileges on db_name.* to user_name@'%' identified by 'password';
> flush privileges;
> exit
```

## 3. Set nginx

```
$ vi /etc/yum.repo.d/nginx.repo
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/\$releasever/\$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
$ yum clean
$ yum info nginx
$ yum install nginx
$ systemctl enable nginx
> Created symlink from /etc/systemd/system/multi-user.target.wants/nginx.service to /usr/lib/systemd/system/nginx.service.
$ systemctl start nginx
```

## 4. Add user&group

```
$ groupadd group_name
$ grep group_name /etc/group
$ useradd -g group_name -s /sbin/nologin -p 'password' user_name
$ echo 'password' | passwd --stdln user_name
$ cat /etc/passwd
```

## 5. Create django project
```
$ cd /home/user_name
$ mkdir main_project
$ cd main_proejct
$ pip3 install virtualenv
$ virtualenv myvenv
$ source myvenv/bin/activate
$ mkdir dir1 dir2
$ cd dir1
$ django-admin.py startproejct sample
$ cd sample/sample
$ vi settings.py
```

```
ALLOWED_HOSTS = ['your_domain', 'your_ip']
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'db_name',
        'USER': 'user_name',
        'PASSWORD': 'password',
        'HOST': '',
        'PORT': '',
    }
}
STATIC_URL = '/static/'
STATIC_ROOT = "/var/www/your_domain/static/"

MEDIA_URL = '/media/'
MEDIA_ROOT = "/var/www/you_domain/media/"
```

```
$ python3.6 manage.py makemigrations
$ python3.6 manage.py migrate
$ python3.6 manage.py createsuperuser
```

## 6. Install uwsgi

```
$ sudo -H pip3 install wheel
$ sudo -H pip3 isntall uwsgi
```

## 7. Install & Set nginx

```
$ cd /etc/nginx/conf.d/nginx.conf
$ mv default.conf default.conf.ogirinal
$ find / -name uwsgi_params
$ cp /etc/nginx/uwsgi_params /home/user_name/main_dir/dir1/project_dir/uwsgi_params
$ vi naignx.conf
```

```
upstream django {
   server unix:/home/user_dir/main_dir/dir2/uwsgi.sock;
}
server {
        listen 80;
        server_name your_domain_or_ip;
        charset utf-8;
        # location = /favicon.ico { access_log off; log_not_found off; }
        location /static/ {
                alias /var/www/your_domain_or_ip/static/;
		# alias /your_project_folder/static/;
        }
        location / {
            include /home/user_dir/main_dir/dir1/proejct_dir/uwsgi_params;
            uwsgi_pass      django;
        }
}
```

## 8. Set uwsgi
There are two modes in uwsgi: Emperor mode and Default(General) mode

The Emperor mode is to run server as a whole computer is one server

Contrary, The Default(General) mode is to run server as django applications are each server

The uWSGI program(both modes) also supports the UDS method

### Emperor mode


```
$ mkdir -p /etc/uwsgi/vassals
$ sudo vi etc/uwsgi/vassals/uwsgi.ini
```

```
[uwsgi]
uid = user_name
base = /home/%(uid)/main_dir
root=%(base)/dir1/project_dir
home = %(base)/myvenv
chdir = %(base)/dir1/project_dir
uwsgi-file = %(chdir)/app_dir/wsgi.py
module = app_dir.wsgi:application
env = DJANGO_SETTINGS_MODULE=app_dir.settings
callable = application
master = True
processes = 5
socket = %(base)/dir2/uwsgi.sock
;for UDS(Unix Domain Socket)
;Default mode also can use UDS 
chmod-socket = 666
vacuum = True
pidfile = /tmp/emperor.pid
deamonize = /home/%(uid)/main_dir/dir2/emperor.log
```
This is Emperor mode

If you need to deploy a big number of apps on a single server, or a group of servers, the Emperor mode is just the ticket.

Start command: `sudo uwsgi --emperor /etc/wusgi/vassals --uid user_name gid group_name`

If you have emperor_wusgi.ini file,

Start command: `sudo nginx -s reload && uwsgi --emperor /etc/uwsgi/vassals --uid user_name --gid group_name`

If you feel like Start command's options are too many and command is too long, then you can make *.ini file about Emperor process

```
$ cd /etc/uwsgi
$ vi uwsgi_emperor.ini
```

```
[uwsgi]
emperor = /etc/uwsgi/vassals
uid = user_name
gid = group_name
master =True
pidfile = /tmp/emperor.pid
vacuum = Ture
daemonize = /var/log/uwsgi/emperor.log
```
Start command: `sudo nginx -s reload && uwsgi --ini /etc/uwsgi/uwsgi_emperor.ini`

Stop command: `sudo nginx -s stop && uwsgi --stop /tmp/emperor.pid`

### General(Default) mode

```
$ cd /home/user_name/main_project/dir2/
$ vi uwsgi.ini
```

```
[uwsgi]
uid = user_name
base = /home/%(uid)/main_dir
root /home/user_name/main_dir/dir1/project_dir
home = %(base)/myvenv
chdir = %(base)/dir1/project_dir
uwsgi-file = %(chdir)/app_dir/wsgi.py
module = app_dir.wsgi:application
env = DJANGO_SETTINGS_MODULE=app_dir.settings
callable = application
master = true
processes = 5
pidfile=/tmp/sample_master.pid
socket = %(base)/dir2/uwsgi.sock
chmod-socket = 666
vacuum = true
deamonize = /home/user_dir/main_dir/dir2/sample.log
;This is uwsgi default mode not emperor
;It is possible to create and run config files with general user accounts
;Start or Stop commands are supposed to be executed in project directory
;Start command: uwsgi --ini dir2/uwsgi.ini
;Stop command: uwsgi --stop /tmp/sample_master.pid
```

## 9. Set sftp

```
$ vi /etc/ssh/sshd_config

```

```
# Chagne or type

#Subsystem sftp /usr/libexec/opnssh/sftp-server  
Subsystem sftp internal-sftp

Match Group group_name
    X11Forwarding no
    AllowTcpForwarding no
    ForceCommand internal-sftp
    ChrootDirectory /home/user_dir
Match User user_name
    PasswordAuthentication yes
    ChrootDirectory /home/user_dir
```

```
$ chown -R user_name.group_name /home/usr_name
$ chown root.group_name /home/user_name
$ chmod -R 755 /home/user_name
$ systemctl start sshd
$ cd /home
$ ls -al
> drwxr-xr-x   3 root group 4096 Sep  4 14:45 user_name(directory)
```

## 10. Run server

```
$ cd /home/user_name/main_dir/dir1/project_dir
$ python3.6 manage.py collectstatic
$ deactivate
$ systemctl restart sshd
$ systemctl restart nginx
$ uwsgi --ini /etc/uwsgi/uwsgi_emperor.ini
$ sudo tail -30 /var/log/uwsgi/emperor.ini
```
