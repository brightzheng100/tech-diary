# MySQL

## Install MySQL on Mac

Install MySQL v5.7 at Mac with remote access granted:

```text
$ brew install mysql@5.7
$ brew services start mysql@5.7

$ echo 'export PATH="/usr/local/opt/mysql@5.7/bin:$PATH"' >> ~/.zshrc
$ mysql -V
$ mysqladmin -u root password 'Password1'

$ vi /usr/local/etc/my.cnf
---
# Default Homebrew MySQL server config
[mysqld]
# Only allow connections from localhost
#bind-address = 127.0.0.1
bind-address = 0.0.0.0
---

$ brew services restart "mysql@5.7"

$ mysql -u root -p
mysql > use mysql;
mysql > CREATE USER 'myuser'@'%' IDENTIFIED BY 'mypasswd';
mysql > GRANT ALL ON *.* TO 'myuser'@'%';
mysql > flush privileges;

$ brew services stop "mysql@5.7"
```

