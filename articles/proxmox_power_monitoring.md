---
title: "Proxmox + Grafana + InfluxDB + TelegrafでProxmoxクラスタのCPU消費電力監視してみた話"
emoji: "⚡️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["proxmox", "grafana", "influxdb", "linux", "ubuntu"]
published: true
---
# はじめに
「Proxmoxクラスタの各ホスト毎のCPU消費電力をProxmox自体のメトリクスと合わせてGrafanaでメトリクス表示をできるようにした」というのを記した記事です。

ほんの数日前にProxmoxを触り始めた「ド初心者」がAmazon Q DeveloperやGeminiと協力して完成した機能なので信頼性は皆無だと思われます。へぇ〜こういうのやったんだ。くらいの感覚でご覧ください。

# なんでやったの？
サーバーを扱っている人間なら誰しも思っているはず。

**数字がたくさんあって動くのは楽しい**

特に最近ProxmoxVEに移行し、VM・LXCをたくさん立ち上げられるようになり、色々動くのを見ることができるようになった私は"欲"にまみれてしまいました。

そんな"欲"に耐えきれずAIと意気投合して構築してしまった人間のお話です。

# 目次

- [Grafana + InfluxDB](#grafana--influxdb)
- [Telegraf（本題）](#telegraf-本題)

# Grafana + InfluxDB
ひとまずメトリクスを表示したいということで**Grafana + InfluxDB**でメジャーなメトリクス表示環境を構築しました。

## 構築
Grafanaでデータを表示するにはData Sourcesが必要です。結構いろいろなデータを使用して表示することができるみたいですが、今回はProxmoxクラスタで使用するので**InfluxDB**を選択しました。

![grafana_datasources](/images/proxmox_power_monitoring/grafana_datasources.webp)

### InfluxDBって？
時系列データベースと呼ばれるDBです。メトリクス、運用監視、センサーデータやリアルタイム分析などに適しており、Grafanaとは相性が良いため広く使われています。

### InfluxDB + Grafanaのインストール
これが正解なのかは不明ですが、ProxmoxVE上に**Ubuntu Server 24.04**のLXCコンテナを作成し、その中でInfluxDBとGrafanaのコンテナを稼働させることにしました。

以下のようなdocker-compose.ymlファイルを作成し、そのままコンテナを作成すればインストールは終わりです。簡単ですね。

```yaml
version: "3"

services:
  influxdb:
    image: influxdb
    container_name: influxdb
    restart: always
    volumes:
      - ./influxdb:/var/lib/influxdb2
    ports:
      - 8086:8086
      - 8083:8083

  grafana:
    image: grafana/grafana
    container_name: grafana
    user: "root"
    restart: always
    volumes:
      - ./grafana:/var/lib/grafana    
    depends_on:
      - influxdb
    ports:
      - 3000:3000

```

### InfluxDBの設定
今回の場合LXCコンテナ上で立ち上げたので、CT作成時に設定したIPアドレスを使用すれば大丈夫です。

```
InfluxDB: http://設定したIPアドレス:8086
Grafana: http://設定したIPアドレス:3000
```

InfluxDBにはじめてアクセスするとOrganizationとBucketの設定画面が出てくると思いますが、どちらもわかりやすいものを入力してください。

![influxdb_setup](/images/proxmox_power_monitoring/influxdb_setup.webp)

Continueをして進んだ先の画面で表示される**Token**は忘れずどこかにメモをお願いします。
**Token**はInfluxDBでのroot権限のようなものなので必ず**安全な場所**に保管してください。

![influxdb_setup_token](/images/proxmox_power_monitoring/influxdb_setup_token.webp)
※これはローカルで作ったテスト用なので大丈夫です。

ひとまずInfluxDBはこれで使用可能になりました。

```
今後必要な情報
- Organization名
- Bucket名
- Token (安全な場所に保管)
```

## Proxmoxの設定
ProxmoxにInfluxDBを接続していきます。

ProxmoxのWebUIにログインし、データセンターに移動してください。<br>
`メトリックサーバ　→　追加　→　InfluxDB`

![proxmox_influxdb](/images/proxmox_power_monitoring/proxmox_influxdb.webp)

```
名前：適当なわかりやすいやつ（例：InfluxDB）
サーバ：先ほどアクセスしていたInfluxDBのアドレス
ポート：8086
プロトコル：今回はHTTP
組織：InfluxDBで設定したOrganization名
Bucket：InfluxDBで設定したBucket名
トークン：先ほどメモしたToken
```
![proxmox_influxdb_create](/images/proxmox_power_monitoring/proxmox_influxdb_create.webp)

Proxmoxで設定するのはこれで終了です。簡単ですね。

## Grafanaの設定
Grafanaにアクセスするとログインを求められますが、初期設定は`ユーザー名：admin, パスワード:admin`です。その後自分でパスワードの再設定ができるので設定してください。

ログインができたら`Connections => Data sources => Search: InfluxDB`で画像のようにInfluxDBを選択してください。
![grafana_influxdb_setup_1](/images/proxmox_power_monitoring/grafana_setup_influxdb_1.webp)

先に進むと設定画面が出てきます。

URLは`http://influxdb:8086`にしてください。今回はDockerで動かしているので内部で接続されます。

それ以外の設定項目は先程と同じような感じで入力してください。
**Query Languageは必ずFluxにするようにしてください。**

![grafana_influxdb_setup_2](/images/proxmox_power_monitoring/grafana_setup_influxdb_2.webp)

設定が終われば`Save&Test`で完了です。

### ダッシュボード
Grafanaでメトリクスを表示するのはDashboardから行います。`Dashboards => + Create dashboard`からダッシュボードを作成してください。

今回は [grafana.com](https://grafana.com/grafana/dashboards/) にあるテンプレートのようなものから簡単にセットアップしたいので開いていただき、`Search dashboards`に`Proxmox`と入力すると色々出てきますが、今回InfluxDBを使用しているためその表記があるものを選んでください。開くと`Copy ID to clipboard`があるので、それからIDをコピーしてください。

IDをコピーできたらGrafanaに戻り、`Import a dashboard`でIDを入力、LoadしてJsonが出てきたらまた下の方のLoadを押すことでダッシュボードの作成ができます。Data sourceは先ほど設定したInfluxDBを選択してください。

![grafana_import_dashboard](/images/proxmox_power_monitoring/grafana_import_dashboard.webp)

`Proxmox => InfluxDB => Grafana`のデータ接続がうまくできていればひとまずGrafanaでのメトリクス表示は完成です。お疲れ様でした。

# Telegraf（本題）
さて、ここまではほとんど [参考文献](#参考にさせていただいた文献) にある記事の内容でしたが、ここからが実質本題です。（ここまで長すぎ）

調べてみたところ、CPUの種類（AMD, Intel）やそもそものシリーズによって消費電力の取得が可能かどうかがバラバラであり、大変そうでした。

しかし、RAPL（Running Average Power Limit）というインターフェースがIntel、AMD両方で使用できそうだったため、今回これを採用することになりました。

https://github.com/kurazuuuuuu/grafana-cpu-power-monitoring

今回の方法でのインストールは全てこのリポジトリにある`setup-cpu-power-monitoring.sh`に実行権限を与えて実行すれば完了します。途中でInfluxDBのIPアドレスやTokenなどの情報が求められるのでそれを入力すればOKです。

**注意：サーバーの安全性は保証できません。自己責任でお願いします。**
**もともとTelegrafを使って何かをしている場合は想定していません。おそらく上書きされます。**

スクリプトで行っている内容は以下の通りです。
```
- CPUの判別
    - アーキテクチャの判別

- 必要パッケージインストール

- Telegrafの設定
    - 電力監視スクリプトの作成
    - Telegrafの設定ファイルの編集（要入力）
    - Telegraf起動確認

- 最終確認
```

## Grafanaへの反映
README.mdにも記載しておりますが、基本的には`Dashboards => Dashboard（作成したもの）=> Edit => Add（Visualization）`まで行き、クエリをREADMEからコピーすれば動作するはずです。

![grafana_result](/images/proxmox_power_monitoring/grafana_result.webp)

Visualizationの設定はGoogleで検索して自分の好きな見た目に設定するのがおすすめです。`Standard options => Unit`を`Watt（W）`にするとちゃんとワット表記になるのでこれはマストかも。

## 詰まってしまった部分
- 当初は`lm-sensors`を使用して消費電力を取得して、それをTelegraf => InfluxDB => Grafanaで表示しようとしましたが、自宅鯖環境ではlm-sensorsを使用して消費電力取得ができないことが判明し、試行錯誤を繰り返した結果`RAPL`にたどり着きました。

- つながりはするものの`0W`表示になってしまう問題がだいぶ続きました。RAPLでデータ取得できるようになったものの、そもそも仕組み上Telegrafの権限ではRAPLのファイルを読み込むことができなかったため、udevを使用して権限設定を行うようにしました。

- RAPLに辿り着く前はCPU使用率から消費電力を推定するというなんとも言えない精度のものだった

# まとめ
- Proxmoxクラスタ全体の消費電力を監視したい
    - Proxmox上にInfluxDB + Grafanaを動かす環境を構築
    - ProxmoxホストでTelegrafを使用してRAPLから消費電力を取得
    - Grafanaでメトリクスを表示

今回の構成により、Proxmoxクラスタの各ホストのCPU消費電力をリアルタイムで監視できるようになりました。数字が動くのを見るのは本当に楽しいですね。

## 備考
- Amazon Q Developerを使用してスクリプトの作成を行いました。
- この記事はAIを使用して文章校正を行っています。

# 参考にさせていただいた文献
https://qiita.com/rokuosan/items/a378e46a89d31d544d4d#influx-%E3%81%AE%E8%A8%AD%E5%AE%9A%E3%82%92%E8%A1%8C%E3%81%86
