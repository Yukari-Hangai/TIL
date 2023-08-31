# バッチ処理
## 「バッチ」とは
複数のプログラムの連続的な連なりによって導き出される結果をひとまとめにしたものがバッチ（Batch）です。
以下は例です。

* プログラムA：データの抽出
* プログラムB：データの加工
* プログラムC：加工データの出力（保存）

プログラムA〜Cによるプログラム群をまとめたものをバッチと言います。
よく定時処理をバッチ処理と表現することがありますが、バッチ処理は定時処理ということではありません。
バッチ処理はプログラムをまとめたものです。
しかし、バッチ処理は定時処理を行うことがほとんどです。

## なぜバッチ処理が必要なのか
毎日のルーチンワークの中で人の手によって行われるものをバッチ処理としておくことで、コンピューターリソースを使い作業効率を向上することができます。
「即時性のない処理」で大量のデータを扱うものではバッチ処理の導入が適切になります。

### 処理の即時性
例えばコンビニのスタッフとしてシステムを利用することを思い浮かべてみましょう。
お客さんが商品を購入した時に店側はレシートを発行しなければなりません。
この場合は商品の購入という処理に対してレシートの発行という処理は「即時性がある処理」と言えます。

一方で、コンビニの1日の売上を集計するといった時はお客さんが商品を購入するたびに、その日の売上を立てる必要はありません。
店舗として当日10:00から翌日の10:00までを1日の売上とする場合には翌日の10:00になった時に1日の売上を集計して報告します。
この場合は商品の購入という処理に対して1日の売上集計は「即時性のない処理」と言えます。

1日のコンビニの利用者は10人や20人といったものではありません。
1日に約1000人が利用するというデータもあります。
毎日、1日の売上集計をする時に1000人分の売上のデータを計算することは「即時性がなく、大量のデータを扱う」ことですのでバッチ処理を導入することが適切だと言えます。

## バッチ処理の実装例
#### 銀行のバッチ処理
銀行では日中、入出金や記帳などあらゆる取引が行われています。
時間外取引においては大量のデータを蓄え、フォーマットを整えて時間になったら送金するといったバッチ処理が存在します。

#### 業務管理システムのバッチ処理
一言に業務管理システムと言っても業種により扱うデータや運用の流れが違います。
ですが業種を問わず共通する事もあります。

* 月間売上（売上、発注料、経費、利益率、粗利）
* 在庫管理

上に挙げたものは業種を問わずに多くの業務管理システムで扱う項目です。
ひとつ、月間売上を例とするならば締日に日毎の売上データから算出して月間売上データを作成するといったバッチ処理があります。
また、業務システムにおいて最もポピュラーなものです。

## バッチ処理の実装
今回作成するバッチ処理では、アプリケーションを完成させよう 8章【課題：アプリケーションを作成してみよう：基礎編】で作成したBookersを使用します。
Bookersでは本のタイトルと本文を入力して投稿することができました。
今回は投稿したデータを2分おきに全て削除するバッチ処理を実装します。

### 削除プログラムを作成
libの配下にbatchフォルダを作成します。
次に作成したbatchフォルダの中でdata_reset.rbファイルを作成して以下を記述して保存します。

~~~ruby
class Batch::DataReset
  def self.data_reset
    # 投稿を全て削除
    Book.delete_all
    p "投稿を全て削除しました"
  end
end
~~~

次に作成したlib/batch/data_reset.rbが読み込まれるように設定します。

config/application.rbに以下を追記してください。
~~~ruby
class Application < Rails::Application
  :
  :  
  config.paths.add 'lib', eager_load: true
end
~~~
このように記述することで、lib配下のファイルが読み込まれるようになります。

### 削除プログラムが実行されるか確認する
投稿データの削除を確認するために、いくつか投稿してデータを作っておきます。
Cloud9上でアプリケーションフォルダに移動して下記のコマンドを実行してみましょう。
~~~ruby
username:~/environment/bookers $ bundle exec rails runner Batch::DataReset.data_reset
~~~
以下のように出力されるはずです。
~~~ruby
Running via Spring preloader in process [プロセスID]
"投稿を全て削除しました"
~~~
次はcronを利用して削除プログラムを定時処理として実行させます。

### cronとは
cron（クーロン）とは定期的に指定のプログラムを実行するためのスケジューラーであり、UNIX系OSに標準搭載されている常駐プログラム（デーモン）です。
MacOSやLinux系OSもUNIX系OSに分類されるものです。
カリキュラムで使用されるCloud9で構築した環境はAmazonLinuxであり、CentOSをベースとするLinux系OSのひとつです。
#### cronの確認
Cloud9上でcronが起動しているか確認するため下記のコマンドを実行します。
~~~ruby
username:~/environment/bookers $ sudo systemctl status crond
● crond.service - Command Scheduler
   Loaded: loaded (/usr/lib/systemd/system/crond.service; enabled; vendor preset: enabled)
   Active: active (running) since 木 2020-12-10 09:34:51 UTC; 4min 47s ago
 Main PID: 3059 (crond)
   CGroup: /system.slice/crond.service
           └─3059 /usr/sbin/crond -n

:
:
~~~
Active: active (running)...となっていればcronは常駐プログラムとして起動しています。
Active: active (dead)....となっている場合は起動していません。

### crontab
crontabはcronに実行してもらう設定を編集するための管理コマンドであり、cronを利用する時にはcrontabの方が馴染み深いものになります。
#### crontabに実行内容を登録
下記コマンドを実行するとviエディタが開くので編集して保存します。

**注意**： 下記のコマンドは間違えがないように慎重に入力してください。
オプションに-rと指定してしまうとcrontabの設定を削除してしまいます。
キーボードのeとrは隣り合わせのため、注意が必要です。
cronを利用する上ではcrontabをセットで覚えましょう。
~~~ruby
username:~/environment/bookers $ crontab -e
~~~
エディタが開いたら下記を入力して保存してください。
その際、「bookers」の箇所を、ご自身のアプリ名に適切に変更することを忘れないように注意しましょう。
また、viの使い方については「開発スキルアップ > viを学ぼう」を参考にしてください。
~~~ruby
*/2 * * * * /bin/bash -l -c 'cd /home/ec2-user/environment/bookers && bundle exec rails runner Batch::DataReset.data_reset >> log/cron.log 2>&1'
~~~
cronを再起動します。
~~~ruby
username:~/environment/bookers $ sudo systemctl restart crond
~~~
これで2分おきに削除プログラムを実行し、log/cron.logに実行した結果を出力します。
log/cron.logの内容を確認するには以下コマンドを実行します。
~~~ruby
username:~/environment/bookers $ sudo tail -f log/cron.log
~~~
tailコマンドは、指定したファイルの末尾10行を出力して表示し、
-fオプションを付けた際、ファイルの内容が増えた場合には新しい末尾を追加表示し、ファイル末尾を出力し続けます。
処理を終了するには、rails sの時と同じ`Ctrl` + `C`を押します。

2分おきにプログラムが実行されるため、しばらく待つと以下のような実行結果が得られます（以下は2回実行された例です）。
~~~ruby
"投稿を全て削除しました"
Running via Spring preloader in process [プロセスID]
"投稿を全て削除しました"
Running via Spring preloader in process [プロセスID]
~~~
このようにcronを利用してプログラムを定期実行することでバッチ処理を実装することができました。

最後に、cronを停止しておきます。
~~~ruby
username:~/environment/bookers $ sudo systemctl stop crond
~~~

### wheneverを利用しよう
railsのgemにはwheneverというcrontab管理を行ってくれるgemが存在します。
こちらを利用して実装してみましょう。

まず、以下コードをGemfileに追記します。
~~~ruby
:
:

gem 'whenever', require: false
~~~
追記できたら、bundle installを実行しましょう。

次に、以下コマンドを実行します。
~~~ruby
username:~/environment/bookers $ bundle exec wheneverize
~~~
以下のように出力されていれば問題ありません。
~~~ruby
0,2,4,6,8,10,12,14,16,18,20,22,24,26,28,30,32,34,36,38,40,42,44,46,48,50,52,54,56,58 * * * * /bin/bash -l -c 'cd /home/ec2-user/environment/bookers && bundle exec bin/rails runner -e [開発環境] '\''Batch::DataReset.data_reset'\'' >> log/cron.log 2>&1'

## [message] Above is your schedule file converted to cron syntax; your crontab file was not updated.
## [message] Run `whenever --help' for more options.
~~~
ですが、これはscheduleの中身をcrontab用に翻訳してくれるだけで、まだcrontabに登録されたわけではありません。
crontabに反映するためのコマンドを実行します。
~~~ruby
username:~/environment/bookers $ bundle exec whenever --update-crontab
~~~
以下のように出力されれば成功しています。
~~~ruby
[write] crontab file updated
~~~
実際にcrontabに反映しているか確認してみましょう。
~~~ruby
username:~/environment/bookers $ crontab -l
~~~
出力結果は以下のようになります。
~~~ruby
# Begin Whenever generated tasks for: /home/ec2-user/environment/bookers/config/schedule.rb at: 2020-08-07 06:09:41 +0000
0,2,4,6,8,10,12,14,16,18,20,22,24,26,28,30,32,34,36,38,40,42,44,46,48,50,52,54,56,58 * * * * /bin/bash -l -c 'cd /home/ec2-user/environment/bookers1-batch && bundle exec bin/rails runner -e development '\''Batch::DataReset.data_reset'\'' >> log/cron.log 2>&1'

# End Whenever generated tasks for: /home/ec2-user/environment/bookers/config/schedule.rb at: 2020-08-07 06:09:41 +0000
~~~
このように出力されていればcrontabに反映されています。

最後に、cronを起動します。
~~~ruby
username:~/environment/bookers $ sudo systemctl start crond
~~~
tail -fなどを用いてlog/cron.logを確認すると2分おきにログが出力されるので、wheneverを利用してcronが動作していることが分かると思います。

確認が終わったら、再度cronを停止しておきましょう。
~~~ruby
username:~/environment/bookers $ sudo systemctl stop crond
~~~~
