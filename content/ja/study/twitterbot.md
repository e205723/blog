---
title: "Twitterのbotを作る+nohupコマンドでdaemon「っぽく」させる"
description: とりあえずnohupは逃げ
publishdate: 2021-05-10T00:08:35+09:00
---

## Twitterのbotを作りたい

B1の頃から、Twitterのbotやりたいなぁ...。って思ってて、最近ちょっと暇だったので作ってみました。

未来、賢くなったら、「馬鹿だなぁ自分」って感じでこの記事を振り返りたい。未来、後輩から多少尊敬されるようになったら、後輩達にこの記事を書いた馬鹿な先輩の足跡を見て多少安心してもらえると嬉しいどす。

### そもそもTwitterbotなんか作って何を学びたいの?

結果から言うと全部逃げの選択を取ってしまって、なんか動くけど...うん、君もっと頑張ろうよ...。ってなる感じのプロダクトになりました。特にセキュリティ関連の脅威がなくても、まあ動くなら正義じゃん?ってな感じで始めるしかない。とりあえず作る過程で何を学びたいか列挙する。

{{< li >}}
- API
  - `A`pplicationが別の`P`rogram(ing)を使うための`I`nterface(仲介役)みたいな
  - いや、こんな説明だとOSの授業で先生に怒られるのは必至
  - API key(API使えるようになるための暗号)ってのが必要になるらしい
- Daemon
  - デェ↑モンって読みます。紀元前98038年に爆誕した(おい)
  - メインメモリ上で常駐(ずっと稼働してる)プログラムのこと
  - バッググラウンドプロセスの一種
{{< /li >}} 

まあ、つまりTwitterのAPIの機能(ツイートをゲットしたり、ツイートを送信したり)を使うプログラムを、daemon化して、常駐化、つまりずっと稼働している状態にしたいっていう野望がありました。

## では早速API keyをゲットしよう

Twitter APIを使う準備をしましょう。

**必要なもの**

{{< li >}}
- 携帯番号、メールアドレスと連携しているTwitterアカウント
- サーバ(学科VMとか)
{{< /li >}}

### ウェブ上でAPI keyを申請する

API keyをゲットするために色々申請しなければなりません、作業は1時間もかからないと思います。

[このサイトに飛んでください](https://developer.twitter.com/en/portal/petition/use-case)

<img src="/images/twitterbot/1_twitterbot.png" alt="1_twitterbot.png">

↑ 僕は趣味でbotを作るので`Hobbyist`を選びました

<img src="/images/twitterbot/2_twitterbot.png" alt="2_twitterbot.png">

↑ 作るのはbotなので`Making a bot`を選択

<img src="/images/twitterbot/3_twitterbot.png" alt="3_twitterbot.png">

↑ 適当に情報を埋めていく

<img src="/images/twitterbot/4_twitterbot.png" alt="4_twitterbot.png">

↑ 英語で「TwitterのAPIやデータをどのように使うつもりなのか」を英語で記述して説明する。コピペはダメかも。

<img src="/images/twitterbot/5_twitterbot.png" alt="5_twitterbot.png">

↑ ここも多少英語で書き書き

<img src="/images/twitterbot/6_twitterbot.png" alt="6_twitterbot.png">

↑ 情報の確認的な

<img src="/images/twitterbot/7_twitterbot.png" alt="7_twitterbot.png">

↑ ポリシー的なのを読んでチェック、なんかあったら自己責任でお願いします

<img src="/images/twitterbot/8_twitterbot.png" alt="8_twitterbot.png">

↑ Twitterアカウントと連携しているメールアドレスへ確認メールが送られる

<img src="/images/twitterbot/9_twitterbot.png" alt="9_twitterbot.png">

↑ メールの受信ボックスへ行って、確認メールを見る。`Confirm your email`をクリック

<img src="/images/twitterbot/10_twitterbot.png" alt="10_twitterbot.png">

↑ [ここのサイト](https://developer.twitter.com/en/portal/dashboard)から上のようなページを探す、そして`Create Project`をクリック

<img src="/images/twitterbot/11_twitterbot.png" alt="11_twitterbot.png">

↑ 適当にプロジェクト名を記入

<img src="/images/twitterbot/12_twitterbot.png" alt="12_twitterbot.png">

↑ `Making a bot`を選択

<img src="/images/twitterbot/13_twitterbot.png" alt="13_twitterbot.png">

↑ プロジェクトを英語で説明

<img src="/images/twitterbot/14_twitterbot.png" alt="14_twitterbot.png">

↑ `Create a new App instead` を選択

<img src="/images/twitterbot/15_twitterbot.png" alt="15_twitterbot.png">

↑ Appの名前を記入

<img src="/images/twitterbot/16_twitterbot.png" alt="16_twitterbot.png">

↑ API keyとかもろもろ表示されるのでメモする、他の人に共有してはいけない

<img src="/images/twitterbot/17_twitterbot.png" alt="17_twitterbot.png">

↑ メモしてねって指示が出される、保存したら`Yes, I saved them`を選択

<img src="/images/twitterbot/18_twitterbot.png" alt="18_twitterbot.png">

↑ 鍵マークをクリック

<img src="/images/twitterbot/19_twitterbot.png" alt="19_twitterbot.png">

↑ Access Token and Secretの`Generate`をクリック

<img src="/images/twitterbot/20_twitterbot.png" alt="20_twitterbot.png">

↑ さっきのAPI keyと同じ感じで、メモして`Yes, I saved them`をクリック

### 注意

このAPI keyは初回発行時はRead Only(Twitterからデータを取得するだけ)の設定なので、ツイートするようにしたければ、設定するページを探して設定を変えて再度API keyもろもろ`Regenerate`してください。

---

## Pythonスクリプトを書いてbotを作成する

### どんなbotを作るかの説明から

`@ojisankobun_bot ojichat at <user_id>`って感じでツイートすると、ツイッターのidが<user_id>の人に

```
@<user_id> <ユーザーネーム>ちゃん、ヤッホー😃✋😊😘😂何してるのかい😜⁉️🤔おじさんはさっきお風呂入ったよ😚🎵💗😍<ユーザーネーム>ちゃんとお風呂いきたいなー(^o^)❗😄ナンチャッテ😚😂😃😘
```

こんな気持ち悪いメンション付きツイートを送信するのを目指す。

### どうやってそんな気持ち悪い文字列を生成するか

[ojichatというCLI](https://github.com/greymd/ojichat)を使います。え、CLI?? ojichatはAPIもあるよ。ってなる感じですが、APIメンドクサイ。APIを学ぶために作っているのにAPI使わないという暴挙...。

### 肝心のTwitter APIはどうする

Tweepyっていうライブラリを使います。

は???

お前。。。この期に及んでライブラリに頼るのかよ!!!!!

ってなりますよね。逃げるのが得意なのです。

### ソースコード

まあ、とりあえずソースコード見てくださいな

**************************************************ってとこは自分のAPI keyを入力してください。

```
#!/usr/bin/python3

import tweepy
import datetime
import os
import time

"""

this bot send a ojichat to a person whose name is written when this bot is mentioned in the following format:

@ojisankobun_bot ojichat at <user_id>

<user_id> does not have to include an '@' symbol.

"""

# information to create an authorization object
API_KEY = '**************************************************'
API_SECRET = '**************************************************'
ACCESS_TOKEN = '**************************************************'
ACCESS_TOKEN_SECRET = '**************************************************'

# create an authorization object
auth = tweepy.OAuthHandler(API_KEY, API_SECRET)
auth.set_access_token(ACCESS_TOKEN, ACCESS_TOKEN_SECRET)

# api object
api = tweepy.API(auth, wait_on_rate_limit=True)

# current_date
current_date = datetime.date.today()

# last_mention_id
last_mention_id = 0

# dictionary of users not to take orders from
users_dict = {}

# test flag
test = True

# fucntion which runs in while true loop
def ojichat_at_designated_user():
    global api
    global current_date
    global last_mention_id
    global users_dict
    global test

    # if date has changed, clear the users_dict dictionary
    if current_date != str(datetime.date.today()):
        users_dict.clear()
        current_date = str(datetime.date.today())

    # get the 20 most recent tweets which mention this bot
    timeline = api.mentions_timeline(count = 20)

    # send ojichats to the people whose names are written in the fetched tweets
    for status in reversed(timeline):

        # if an error is caught during this proccess, add a key to the users_dict dictionary
        try:
            # if the number of sender's tweets has reached the maximum number, skip the current proccess
            if len(users_dict[status.user.screen_name]) >= 5:
                continue
        except KeyError:
            # add a list as an element paired with the key in the dictionary
            users_dict[status.user.screen_name] = []

        # if the id of the fetched tweet is newer than the last one, the number of tweets from the same sender has not reached the maximum number of times, and the tweet includes a string " oji at ", excecute the following proccesses
        if int(status.id_str) > last_mention_id and not status.id_str in users_dict[status.user.screen_name] and " ojichat at " in status.text:
            # overwrite last_mention_id
            last_mention_id = int(status.id_str)
            # add the tweet id to the list paired with the screen name key in the users_dict dictionary
            users_dict[status.user.screen_name].append(status.id_str)
            
            # get the user_screen_name which is written on the tweet
            user_screen_name = status.text.split(" ojichat at ")[1]

            # when an error is caught during the following processes, skip the current proccess
            try:
                user_name = api.get_user(user_screen_name).name
                # get an ojichat from cli using "name" as an argument and replace "name" with the user_name
                output = os.popen('ojichat name').read().replace('name', user_name)
                if output != "":
                    if(not test):
                        # tweet an ojichat addressing the user_screen_name at the beginning of the tweet
                        api.update_status('@' + user_screen_name + ' ' + output)
                    else:
                        # for test
                        # print('@' + user_screen_name + ' ' + output)
                        pass
            except:
                continue
    else:
        test = False
    
    # wait for 12 seconds
    time.sleep(12)

if __name__ == '__main__':
    while True:
        ojichat_at_designated_user()
``` 

### 解説

コードの説明がめんどくさい。

とりあえず大事なところだけ

```
user_name = api.get_user(user_screen_name).name
# get an ojichat from cli using "name" as an argument and replace "name" with the user_name
output = os.popen('ojichat name').read().replace('name', user_name)
```
これは、

1. 指定されたツイッターのidからユーザーネームを取得してuser_nameという変数に保存する

2. コマンドで`ojichat name`と叩いた結果を文字列として取得
```
nameちゃん、ヤッホー😃✋😊😘😂何してるのかい😜⁉️🤔おじさんはさっきお風呂入ったよ😚🎵💗😍nameちゃんとお風呂いきたいなー(^o^)❗😄ナンチャッテ😚😂😃😘
```
みたいなのが得られる

3. この文字列のnameをuser_nameが保存している値で置換する
```
@<user_id> <ユーザーネーム>ちゃん、ヤッホー😃✋😊😘😂何してるのかい😜⁉️🤔おじさんはさっきお風呂入ったよ😚🎵💗😍<ユーザーネーム>ちゃんとお風呂いきたいなー(^o^)❗😄ナンチャッテ😚😂😃😘
```
みたいにするって処理なんですけど

え?わざわざ置換するの?
```
user_name = api.get_user(user_screen_name).name
output = os.popen('ojichat ' + user_name).read()
```
ってやってもええやん、となっちゃう人もいると思います。

なんですけど、これはCLIインジェクション攻撃を食うセキュリティホールになります。

たとえば、対象のidから取得したユーザーネームが`name & rm -rf *`みたいな名前だと

```
output = os.popen('ojichat ' + user_name).read()
```
↑ このプログラムによって`ojichat name & rm -rf *`っていうコマンドが叩かれる。

怖いね。。。

だから代用策で置換していくって感じに書きました。

`while True:`構文と`time.sleep()`関数を使って、休み休み、永遠と稼働するPythonを作りましてですね。あとはどっかのサーバーに置いていく感じになります。

## nohupコマンドでdaemonもどきを作る

サーバーにsshしてさっきのスクリプトを置いて、nohupコマンド叩いてください。

`nohup oji_tweet.py &`

みたいにお願いします。

プロセスの確認は

`ps -aux | grep ojj_tweet`とかで調べるとでます。

停止したい時はkillコマンドを使用してください。

## 完成

**できたbotが[こちら](https://twitter.com/ojisankobun_bot)**
