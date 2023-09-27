## DBeaverでMySQLにアクセスできない問題とその解決策

* DBeaverでMySQLにアクセスしようとしたが、Access Deniedエラーが出て接続できない。

### 調査
* ps aux | grep mysqldでプロセスを確認。
* ローカルとDocker内の両方でMySQLが起動していることを発見。

### 原因
* ローカルとDockerでMySQLが同時に起動していた。
* mysqldがmysqld_safeスクリプトを介して実行されていた。
* mysqld_safeは、プロセスが死んだ場合に自動でmysqldを再起動する。

アドバイス①： ローカルでMySQLが起動していると、そのMySQLに誤って接続し、ユーザー情報が不足してaccess deniedになるケースが多い。

### 対処法
* sudo kill -9 [プロセスID]でmysqld_safeプロセスを強制終了。
* brew services stop mysql@8.0でHomebrewで管理しているMySQLサービスを停止。

アドバイス②： HomebrewはlaunchdにMySQLを登録してしまう可能性がある。そのため、PCを再起動すると同様の問題が再発するかもしれない。 brew services listで確認し、ステータスがnoneなら問題なし。

### 確認方法
* MySQLが停止しているか確認。bash  Copy codelsof -i:3306   
* PC再起動時の自動起動設定を確認。bash  Copy codebrew services list   
* ステータスがnoneなら自動起動はしない。
