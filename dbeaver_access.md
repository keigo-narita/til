# DBeaverでMySQLにアクセスできない問題とその解決策

* DBeaverでMySQLにアクセスしようとしたが、Access Deniedエラーが出て接続できない。
* （「Public Key Retrieval is not allowed」については、「allowPublicKeyRetrieval」をtrueにすると解決できた）

## 調査
* `ps aux | grep mysqld`でプロセスを確認。
```zsh
ps aux | grep mysqld
```
* ローカルとDocker内の両方でMySQLが起動していることを発見。
```
app % lsof -i:3306
COMMAND     PID USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
mysqld    62673 test   20u  IPv4 0xe68cc4c6808f0f65      0t0  TCP localhost:mysql (LISTEN)
com.docke 66453 test  139u  IPv6 0xe68cc4c1b5e16c4d      0t0  TCP *:mysql (LISTEN)
```

## 原因
* ローカルとDockerでMySQLが同時に起動していた。
* mysqldがmysqld_safeスクリプトを介して実行されていた。
* mysqld_safeは、プロセスが死んだ場合に自動でmysqldを再起動してしまう

> アドバイス①： ローカルでMySQLが起動していると、そのMySQLに誤って接続し、ユーザー情報が不足してaccess deniedになるケースが多い。

## 対処法
* `sudo kill -9 [プロセスID]` でmysqld_safeプロセスを強制終了。
* ` brew services stop mysql@` でHomebrewで管理しているMySQLサービスを停止。
```zsh
brew services stop mysql@`
```

> アドバイス②： HomebrewはlaunchdにMySQLを登録してしまう可能性がある。そのため、PCを再起動すると同様の問題が再発するかもしれない。 `launchctl mysql plist 削除`みたいな単語でぐぐってみて、PC再起動しても mysql が起動しないように設定しておくことをおすすめ

## 確認方法
* MySQLが停止しているか確認。
```zsh
lsof -i:3306
``` 
* mysqldがどのプロセスによって起動されているか確認する
```zsh
ps aux | grep mysqld
```
* `brew services list` で確認し、ステータスが `none` なら問題なし。
```zsh
brew services list
```
- - -
以下、実際のコマンド。
```
app % ps aux | grep mysqld

test     62901   0.8  0.0 407962208    160 s022  S+    7:15PM   0:00.00 grep mysqld
test     44379   0.0  0.0 408530160    704   ??  S    Wed12PM   0:00.06 /bin/sh /opt/homebrew/opt/mysql@8.0/bin/mysqld_safe --datadir=/opt/homebrew/var/mysql
test     62673   0.0  0.2 410317968  28432   ??  S     7:13PM   0:00.73 /opt/homebrew/opt/mysql@8.0/bin/mysqld --basedir=/opt/homebrew/opt/mysql@8.0 --datadir=/opt/homebrew/var/mysql --plugin-dir=/opt/homebrew/opt/mysql@8.0/lib/plugin --log-error=local.err --pid-file=k5mbp.local.pid
app % sudo kill -9 44379

Password:
app % brew services stop mysql@8.0

Stopping mysql@8.0... (might take a while)
==> Successfully stopped mysql@8.0 (label: homebrew.mxcl.mysql@8.0)
app % lsof -i:3306
COMMAND     PID USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
com.docke 66453 test  139u  IPv6 0xe68cc4c1b5e16c4d      0t0  TCP *:mysql (LISTEN)

% brew services list

Name        Status User File
jenkins-lts none
mysql@8.0   none
php         none
php@8.0     none
php@8.1     none
```

この後、DBeaverに戻って接続すると、無事にDBにアクセスできた。

## 参考
* https://qiita.com/ymzkjpx/items/449c505c50ee17b6e8f9
* https://mykii.blog/brew-stop-local-mysql/
* https://qiita.com/yukihigasi/items/a1c84b14ad6807896003
