# isucon7q

## 作業ログ
https://hackmd.io/EzK-6jVtQdGN-1DO6gaY0w

## 手順やコマンドなどのメモ
### isuconユーザに変更
$`su - isucon`

### ユーザ周り
- ユーザ作成
$`sudo adduser <ユーザ名>`
- ユーザの確認
$`cat /etc/passwd | grep <ユーザ名>`

- エラー「`sudo: unable to resolve host <外部IP>`」

  対策方法：https://qiita.com/naberina/items/aa85831d863d7341c93f

### systemctl
サービスは`Unit`という単位で`/etc/systemd/system`配下にユニット定義ファイルがある
- 起動　$`systemctl start ${Unit}`
- 停止 $`systemctl stop ${Unit}`
- 自動起動有効 $`systemctl enable ${Unit}`
- 自動起動無効　$`systemctl disable ${Unit}`
- ステータス表示 $`systemctl status ${Unit}`
- サービス一覧 $`systemctl list-unit-files --type=service`

### Nginx
#### ログフォーマットを変更する（アクセスログのパスは例）
$`sudo vim <nginxの設定ファイル ex) /etc/nginx/nginx.conf>`
```
http {

        ...

        ##
        # Logging Settings
        ##
        
        ## フォーマット設定を追加
        log_format ltsv "time:$time_local"
                "\thost:$remote_addr"
                "\tforwardedfor:$http_x_forwarded_for"
                "\treq:$request"
                "\tstatus:$status"
                "\tmethod:$request_method"
                "\turi:$request_uri"
                "\tsize:$body_bytes_sent"
                "\treferer:$http_referer"
                "\tua:$http_user_agent"
                "\treqtime:$request_time"
                "\tcache:$upstream_http_x_cache"
                "\truntime:$upstream_http_x_runtime"
                "\tapptime:$upstream_response_time"
                "\tvhost:$host";
                
        ## 自分で設定したフォーマットlog_formatのlstvを使用するよう修正
        access_log /var/log/nginx/access.log　lstv;
        
        ...
        ...
        ...
```
フォーマット設定：https://github.com/tkuchiki/alp/blob/master/README.ja.md#log-format

$`sudo systemctl restart nginx`

### Alpで解析
#### インストール
$`sudo apt install unzip`
$`wget https://github.com/tkuchiki/alp/releases/download/v1.0.3/alp_linux_amd64.zip`
$`unzip alp_linux_amd64.zip`
$`rm -rf alp_linux_amd64.zip`
$`sudo mv alp /usr/local/bin/alp`

#### ベンチマークを実行してalpで結果表示
$`cat <access.logへのパス ex) /var/log/nginx/access.log> | alp ltsv`

### 便利なコマンド
- jq（整形） $`cat <ファイル名> | jq`
- grep（検索） $`cat <ファイル名> | grep <検索文字列>`
