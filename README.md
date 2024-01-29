# パルワールドサーバ

## 利用者向け
### パルワールドサーバー
自動アップデート  

### 接続方法
1. マルチプレイ(専用サーバ)を選択  
![image](https://github.com/hideBBBtec/picnic_palworld/assets/54278033/750bbc9f-ff23-4732-810a-5c4ecc1813c9)

2. IPアドレスとポートを入力して接続（ポートは8211で固定）  
![image](https://github.com/hideBBBtec/picnic_palworld/assets/54278033/72bc54e8-c33e-4891-83d3-a4f566ebeb9a)


### Discord bot利用方法
|スラッシュコマンド|アクション|機能|備考|
|---|---|---|---|
|/palmanage|start|サーバを起動する|実行後パルワールドが起動するまで1,2分待つ|
||status|サーバが起動しているかどうか、およびIPアドレスを確認する|サーバのIPアドレスはこれで確認|
||stop|サーバを停止する|遊んだ後は節約のためにサーバ停止する|


### サーバーが起動しているか確認したいとき
1. Discord botのコマンド「/palmanage status」で現在サーバーが起動しているか確認する
2. サーバが起動していた場合、コマンドの結果でIPアドレスがわかるので接続する
3. サーバが起動していなかった場合、「/palmanage start」で起動する
4. 1,2分で起動するので、「/palmanage status」でIPアドレスを確認して接続する

### セーブデータバックアップ
1時間おきにセーブデータをバックアップ
設定変更とかで余裕でデータファイルぶっとんだとかいう報告があったため

### アップデート
30分おきにパルワールドのアップデートをチェック
再起動が発生する5分前から数回に分けて周知（ゲーム内チャット）

### メモリ監視
30分おきに物理メモリを監視する
物理メモリが80パーセントを超えていた場合、再起動を実行する
再起動が発生する5分前から数回に分けて周知（ゲーム内チャット）


## 開発者向け

### 要件
#### サーバスペック要件
- メモリ16 GB以上が推奨だが、8 GBでスワップ使う  
t3a.large（2 vVPU/8 GB）  
※steamcmdがAmazon Linux 2023と互換してないっぽいので、Amazon Linux2にする

#### 節約要件
- 使ってない間はサーバ停止
- Elastic IP使うとお金かかるから、Public IPは変わってもよいものとする

#### 利便性要件
- Discordからサーバ起動停止できる
- Discordからサーバ状態がわかる（起動しているかどうか）
- 今のサーバのIPアドレスが確認できる

#### 自動アップデート
リリース直後なので頻繁にアップデートが入ると予想
3時間置きにアップデートチェックして、アップデートがあれば自動で再起動する

#### バックアップ要件
要件等

### 実施した手順
#### Palworldサーバ作成
1. EC2サーバ起動
2. SELinuxの無効化
3. Palworldユーザの作成
4. steamcmdのインストール  
5. Palworld Dedicated Serverのインストール
6. Palworld起動用スクリプトを作成し、サーバ起動時に自動起動するように実装
7. 自動アップデートスクリプト仕込み

#### Discord bot作成
1. サイトを参照してアプリケーションを作成
2. botの招待リンクは下記
https://discord.com/api/oauth2/authorize?client_id=1139242338254852209&permissions=0&scope=bot%20applications.commands
3. node実行環境を作成し、プログラム作成
4. スラッシュコマンドをdiscordにデプロイ
5. discord-interactionsのLambda Layerを作成し、スラッシュコマンドを受けるLambda関数を作成（discordBot_minemanage）
6. 上記のLambda関数にAPIGatewayを紐づけて、discordbotのInteraction URLに登録する
7. EC2をそれぞれ起動、停止、状態確認するLambda関数を作成

#### 自動停止スクリプト作成（cron）
1. ログを監視するシェルスクリプトを作成（check_access_to_shutdown.sh）
2. rootユーザーのcronに登録し、10分置きに確認するが、起動後15分間は停止しないようにする

#### ワールド再作成
要調査

#### バックアップ機能作成
未実装

### 参照サイト
- プレイヤー人数に応じた推奨スペックとインストール方法  
https://qiita.com/naoya-i/items/e907a6b949e5da36d532
- ARKのサーバを立てた人の参考サイト
https://zenn.dev/murnana/scraps/51d9847aed8961
- DiscordbotをLambdaで作成する  
https://zenn.dev/nacal/articles/e7f0d481661ec0
- Discordbot作成方法  
https://www.geeklibrary.jp/counter-attack/discord-js-bot/
- discord.js公式Doc  
https://discordjs.guide/slash-commands/deleting-commands.html
- shellでログ解析する方法  
https://qiita.com/moneymog/items/16d2f843c344a5ace51a
- 3秒以上かかる処理に対してdeferする方法
https://dev.classmethod.jp/articles/discord-interaction-endpoint-deferred-multi-lambda/

