# RS作業依頼（本日・USBから/dataへのコピー）

作成日: 2026-10-01
対象: RS端末での作業

## やること

USBを挿して、1つのコマンドを実行するだけです。コピーと内容照合はバックグラウンドで自動的に進みます。実行後はその場を離れて構いません。

**USBは処理が完了するまで絶対に抜かないでください。**

## 所要時間の目安

| 作業 | 所要時間 |
|---|---:|
| ログイン・USB挿入・認識確認 | 1分未満 |
| マウント・コピー先作成 | 1分未満 |
| コピー（約119GiB） | 15分〜2時間程度 |
| 内容照合（SHA-256） | 5分〜20分程度 |
| **合計（バックグラウンド処理込み）** | **最大2〜3時間程度** |

上記のうち「コピー」と「内容照合」はバックグラウンドで実行するため、**待機は不要**です。

## 手順

### 1. ログインしてUSBを挿す

RS画面にログイン後、USBを挿して20秒待ってから、次を実行してください。

```bash
ls -l /dev/disk/by-label/ && df -h /data
```

**確認すること**

- `KIOXIA`という名前が表示される
- `/data`に130GiB以上の空きがある

どちらか片方でも条件を満たさない場合は、ここで止めて結果を共有してください。

### 2. マウントしてコピーを開始する

以下を1つのコマンドとしてそのまま実行してください（コピー先フォルダ作成からコピー、照合まで自動で進みます）。

```bash
mkdir -p /mnt/rs-usb && \
mount -o ro /dev/disk/by-label/KIOXIA /mnt/rs-usb && \
ls /mnt/rs-usb && \
mkdir /data/RS_DELIVERY_rs-20261001 && \
nohup bash -c '
rsync -rt --partial --info=progress2 /mnt/rs-usb/RS_DELIVERY_rs-20261001/ /data/RS_DELIVERY_rs-20261001/
echo "RSYNC_EXIT=$?"
cd /data/RS_DELIVERY_rs-20261001
sha256sum -c evidence/SHA256SUMS --quiet && echo HASH_OK || echo HASH_NG
echo JOB_DONE
' > /data/RS_DELIVERY_rs-20261001.log 2>&1 & disown && \
echo "started pid=$!"
```

**確認すること**

- `RS_DELIVERY_rs-20261001`というフォルダ名が一度表示される（USBの中身一覧）
- 最後に`started pid=数字`が表示される

途中で`mkdir`が「File exists」、マウントが失敗、など何らかのエラーが出た場合は、そこで止めて画面を共有してください。**上書き・削除・再実行はしないでください。**

### 3. 本日の作業はここまで

`started pid=数字`が表示されたら完了です。この画面を閉じても、ログアウトしても処理は止まりません。USBは挿したままにしてください。

## 共有してほしいもの

- 手順1の実行結果
- 手順2の実行結果（特に`started pid=`の行）

## 翌日（構築担当者が）確認する内容

```bash
tail -n 20 /data/RS_DELIVERY_rs-20261001.log
```

`JOB_DONE`と`HASH_OK`が出ていれば、コピーと照合は成功しています。`HASH_NG`が出ている、または`JOB_DONE`が出ていない場合は、ログ全文を共有してください。USBの取り外しは、`JOB_DONE`を確認した後に案内します。
