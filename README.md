# isuconで使うコマンドのメモ

## 作業ログ
https://hackmd.io/EzK-6jVtQdGN-1DO6gaY0w

## isuconユーザに変更
```
su - isucon
```

## ユーザ周り
#### ユーザ作成
```
sudo adduser <ユーザ名>
```

#### ユーザの確認
```
cat /etc/passwd | grep <ユーザ名>
```

#### エラー「`sudo: unable to resolve host <外部IP>`」

対策方法：https://qiita.com/naberina/items/aa85831d863d7341c93f


## systemctl
サービスは`Unit`という単位で`/etc/systemd/system`配下にユニット定義ファイルがある
起動
```
systemctl start ${Unit}
```

停止
```
systemctl stop ${Unit}
```

自動起動有効
```
systemctl enable ${Unit}
```

自動起動無効
```
systemctl disable ${Unit}
```

ステータス表示
```
systemctl status ${Unit}
```

サービス一覧
```
systemctl list-unit-files --type=service
```

## Nginxのログフォーマットを変更する

Nginxの設定ファイルを編集してログフォーマットを変更（アクセスログのパスは例）
```
sudo vim <nginxの設定ファイル ex) /etc/nginx/nginx.conf>
```

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

Nginxを再起動
```
sudo systemctl restart nginx
```

## Alpで解析

レスポンスタイムで解析できる

#### インストール
$`sudo apt install unzip`

$`wget https://github.com/tkuchiki/alp/releases/download/v1.0.3/alp_linux_amd64.zip`

$`unzip alp_linux_amd64.zip`

$`rm -rf alp_linux_amd64.zip`

$`sudo mv alp /usr/local/bin/alp`

#### ベンチマークを実行してalpで結果表示
$`cat <access.logへのパス ex) /var/log/nginx/access.log> | alp ltsv`

## pprofで解析

CPUリソースを消費している箇所を解析できる
https://golang.org/pkg/net/http/pprof/#pkg-overview

pprofを取得
```
go get -u github.com/google/pprof
```

import文とmain関数内に以下を追記
```
import _ "net/http/pprof"

func main() {

  go func() {
    log.Println(http.ListenAndServe("localhost:6060", nil))
  }()
  
}
```

buildして、サービスを再起動

プロファイリングを実行
```
go tool pprof http://localhost:6060/debug/pprof/profile?seconds=60
```

ターミナルをもう1つ立ち上げてベンチマークを実行

pprofの対話モード、または解析結果のgzファイルをpprofコマンドで指定して解析を始める

### pprofの解析コマンド

どの関数が重いのかを見る

`(pprof) top`

```
Flat：関数の処理時間
Flat%：各Flatの全体に対する割合
Sum%：スタック履歴からの累計Flat%
Cum：待ち時間も含めた処理時間
Cum%：各Cumの全体に対する割合
Name：関数名
```

特定の関数に対して、呼び出し元がどこか・どれくらいcallしているかを見る

`(pprof) peek 関数名`

ソースコードの各行がどれくらい時間がかかっているか、どれくらいリソースを使っているか

`(pprof) list 関数名 or パッケージ名`

## 便利なコマンド
jq（整形）
```
cat <ファイル名> | jq
```

grep（検索）
```
cat <ファイル名> | grep <検索文字列>
```
