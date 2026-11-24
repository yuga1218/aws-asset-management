# AWS学習記録

## 1. 基本情報
- 学習日：2026-11-14
- 学習時間：1h
- 学習テーマ：CloudWatchの仕組みを学び、EC2の監視を行得るようになる

## 2. 学習目的
AWS CloudWatchを利用して、EC2インスタンスの稼働状況を監視するとともに、EC2上で稼働するNginxのログをCloudWatch Logsに集約する仕組みを構築する。

## 3. 実施内容
実際に行った作業を記載する。

- EC2インスタンスの「モニタリング」機能を用いて、CPUの使用率やアクセスI/Oの量を確認した
- CloudWatchを操作し、異常検知の閾値などを設定した

- 1.「sudo dnf install amazon-cloudwatch-agent -y」：EC2にCloudWatch Agentというソフトウェアを導入する
- 2.「/opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a status」：インストール状況の確認
- 3.「sudo mkdir -p /opt/aws/amazon-cloudwatch-agent/etc」：設定ファイルを作成する
- 4.「sudo nano /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json」：設定ファイルを編集する
  →設定画面にJSONを入力することができる
- 5.「sudo python3 -m json.tool /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json > /dev/null && echo "JSON OK"」：JSONの構文チェック
- 6.「sudo ls -l /var/log/nginx/」；nginxのログファイルの存在確認
- 7.「sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \-a fetch-config \
    -m ec2 \-c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json \
    -s」：CloudWatch Agentの起動
- 8.CloudWatch→ログ管理→ロググループを確認

## 4. 技術的な理解
- CloudWacth MetricsでCPU使用率をデータとして収集し、CloudWacth Alarmで閾値を設定する
→今回はCPUの使用率が80%以上になった場合に通知が送られるように。
【サービス詳細】
- CloudWatch Metrics：CPU使用率などの数値データを収集・監視するもの
- CloudWatch Logs：EC2やアプリケーションが出力するログを収集・保存・検索するもの

【CloudWatch Agentの利用】
- CloudWatch Agent：CloudWatch Agentは、EC2内部のログやメトリクスを収集し、CloudWatchに送信するためのソフトウェア
- -y：確認に自動でYesと回答するコマンド
- mkdiir：ディレクトリを作成
- -p：途中のディレクトリもまとめて作成する
- nano：Linux上でテキストファイルを編集するためのエディタ

【JSONの利用詳細】
- 「python3 -m json.tool」：JSONの構文をチェックする
- 「> /dev/null」：チェック結果の詳細表示を抑制する
- 「&& echo "JSON OK"」：チェックが成功した場合のみ「JSON OK」と表示する

- 「圧縮（.gz）」：古いログを圧縮して、ディスク容量を節約する仕組み

【Agentの起動詳細】
- fetch-config：設定ファイルを読み込む
- -m ec2：EC2錠で動作させる
- -c file:：使用する設定ファイルを指定
- -s：Agentを起動する

## 5. 設計理由
なぜこの設定・構成を採用したのか。
- nginxのアクセス/エラーログをsession managerだけでなく、CloudWatch Logsで確認できるように設計した
  →AWSコンソールからログを確認できるようにするため
- EC2から必要なサービスへログインするために適切なIAMロールをアタッチした
- Linuxではファイルやディレクトリごとにアクセス権限が制限されており、必要に応じてsudoコマンドを使用した
- nginxのアクセス/エラーログはそれぞれ記録される内容が異なる

## 6. 発生した問題と解決方法
- 発生した問題：アクセス/エラーログがCloudWatchに反映されるまで時間がかかった
- 原因：Agentを起動してからロググループが追加されるまでに一定のラグが発生する
- 対応：数分待った
- 解決結果：アクセスロググループが作成された。一方でエラーログは作成されなかった。理由としてエラーがまだ発生していないため、エラーログが存在していないことが考えられる。

## 7. 学習成果・確認結果
EC2のCPU使用率を監視するCloudWatch Alarmを作成し、CloudWatch Agentを利用したNginxアクセスログのCloudWatch Logsへの集約を実現した。

## 8. 今後の課題
今後は、監視対象の拡張やログを利用した障害調査など、実際の運用を意識した学習を進める。
