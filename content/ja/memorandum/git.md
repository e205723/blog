---
title: "Gitの基本"
description:
publishdate: 2021-05-22T00:30:48+09:00
---

### なぜこれを書いているか

僕はGitあまり深くやっていないので、勉強用にこの記事を書いてます

### この記事の対象者

{{< li >}}
- `git add`, `git commit`, `git push`くらいしかやったことない人
- ただぼんやりとしかGitしたことない人
- `merge`ってなんぞや...って人
{{< /li >}}

### 注意

この記事の内容は「Githubありき」という感じです

Linus Torvaldsさんごめんなさい

## 登場するgitコマンド

基本的に引数は省力します

{{< table >}}
| コマンド | 操作 |
|---|---|
| git init | ディレクトリをGitで使う宣言 |
| git add | ファイル、フォルダをステージングエリアに置く |
| git status | ステージングエリアにあるファイル、フォルダの確認 |
| git diff \-\-cashed | ステージングエリアにあるファイルの前のバージョンとの変更点を表示 |
| git commit | ステージングエリアにあるファイル、フォルダをリポジトリに移動させてバージョンを確定する |
| git commit -m | エディタを開かずにコミットメッセージを引数として書き、git commitを実行 |
| git commit –amend | 直前のコミットを修正する |
| git log | コミットの履歴を表示 |
| git log \-\-oneline | コミットのログを1行だけのコンパクトな形で表示する |
| git log -p | commitしたときの前のバージョンとの差分(変更点)を表示する |
| git log \-\-stat | コミットごとに変更されたファイル名を列挙して表示する |
| git remote | ローカルのリポジトリをリモートリポジトリに登録する |
| git config -l | リモートリポジトリの場所を確認する |
| git push | ローカルリポジトリのコミットをリモートリポジトリに反映させる |
| git reset | 指定されたコミットのバージョンへ戻す |
| git reset \-\-hard HEAD | 直前のコミットへ戻る |
| git reset \-\-hard HEAD^ | 前々回のコミットへ戻る |
| git reset \-\-hard ORIG_HEAD | git resetによってキャンセルされたコミットへ戻る |
| git branch | ブランチの一覧の表示 |
| git branch \<branch name\> | \<branch name\>という名前のbranchを作る |
| git branch -d | 指定されたブランチを消す |
| git checkout | 指定されたのブランチへ移動 |
| git checkout -b | 新しいブランチを作って移動する |
| git merge | 現在のブランチへ指定されたブランチにあるファイル、フォルダを反映する |
{{< /table >}}

## まずはGitで登場する領域から

ここで**ステージングエリア**を含む、4つの重要な領域について勉強しましょう

{{< table >}}
| 領域 | 概念 |
|---|---|
| 作業ディレクトリ | ファイルの作成、修正を行う場所 |
| ステージングエリア | リポジトリに確定する予定のファイルの置き場所 |
| リポジトリ(ローカル) | バージョンが確定したファイルの置き場 |
| リモートリポジトリ | ネットワーク上にあるリポジトリ |
{{< /table >}}

図にするとこんな感じ

<img src="/images/git/1.png" alt="1.png">

Gitで扱う領域について視覚的に理解できたら実際に手を動かしてみましょう

## git init

ディレクトリを作って、移動

```
$ mkdir test
$ cd test 
```

そしたら、ここで`git init`!!

```
$ git init
```

これは 「`test`っていうディレクトリをGitで使うよ」って宣言みたいな感じ

`
Initialized empty Git repository in /Users/foo/bar/test/.git/
`

みたいのが出ればオッケー

## git add

```
$ echo "hoge 1" > hoge.txt
```

で文字列`hoge 1`を一行目に含む`hoge.txt`を作成

そして、このファイルを**作業ディレクトリ**から**ステージングエリア**に置くために`git add`というコマンドを使います

```
$ git add hoge.txt
```

また、「このファイル、新しく生成したり、編集したりしたけど、ステージングエリアに`git add`して置いたっけ...」ってなることがあると思いますが

```
$ git status
```

とすることで、`add`したファイル、`add`がまだされてないファイルの一覧が出ます

作業ディレクトリのファイルを全部`add`したいと思うことがあると思います、そんな時は

```
$ git add .
```

でファイル全部を`add`することができます

`git add .`したら`git status`で確認することも忘れずに!

そして、ファイルがステージングエリアにある状態で`diff --cashed`コマンドを叩くと

```
$ git diff --cashed
```

`add`したファイルの前のバージョンとの変更点が出力されます

ここまでの作業で、下の図の様に`hoge.txt`を**作業ディレクトリ**から**ステージングエリア**に置くことができました

<img src="/images/git/2.png" alt="2.png">

## git commit

では次は、**ステージングエリア**からローカルの**リポジトリ**に**add**したファイルを`commit`してみましょう

```
$ git commit
```

上記のコマンドを実行したら下のようにVimが立ち上がります

<img src="/images/git/3.png" alt="3.png">

`commit`するたびに、作業の内容(追加/削除したファイル、編集したファイルの内容などの変更点)をそこに書きます

なので、立ち上がったVimで`i`で**insert mode**に入って`1st commit`と打ってください

<img src="/images/git/4.png" alt="4.png">

そしたら`esc`を押して`:wq`でVimから抜けてください

```
 1 file changed, 1 insertion(+)
 create mode 100644 hoge.txt
```

こんなログが残されていると思います

ちなみにVimでいちいちメッセージを書くのが面倒だったら

```
git commit -m "1st commit"
```

で`-m`のオプションを付けて`commit`してもいいです

そしたら`commit`した履歴を`git log`コマンドで確認してみましょう

```
$ git log
```

このコマンドを打つと下のように出力されるはずです

```
commit c91ff94f6664388c5032a6466f64e00dac908c50 (HEAD -> master)
Author: hoge hoge <hoo@bar>
Date:   Fri May 14 22:27:40 2021 +0900

    1st commit
```

`git log`には色々なオプションがあるので、機能と一緒に羅列していきます

{{< table >}}
| オプション | 機能 |
|---|---|
| git log \-\-oneline | コミットのログを1行だけのコンパクトな形で表示する |
| git log \-p | commitしたときの前のバージョンとの差分(変更点)を表示する |
| git log \-\-stat | コミットごとに変更されたファイル名を列挙して表示する |
{{< /table >}}

ここまでの作業で下の図まで進めることができました

<img src="/images/git/5.png" alt="5.png">

## git push

ローカルの**リポジトリ**に`commit`したら、**リモートリポジトリ**に`push`する段階に入ります

ではまずは、**リモートリポジトリ**を用意しましょう

<img src="/images/git/6.png" alt="6.png">

上のスクリーンショットのように自分のGithubのリポジトリ閲覧ページへ行って、右上の`New`ボタンをクリック

<img src="/images/git/7.png" alt="7.png">

リポジトリの名前を書く、書いたらスクロールダウン

<img src="/images/git/8.png" alt="8.png">

`Create repository`ボタンをクリック

<img src="/images/git/9.png" alt="9.png">

リポジトリが作られたら、この画面に飛びます

ローカルの**リポジトリ**を作られた**リモートリポジトリ**に登録するには、`git remote`コマンドを使います

下線部をコピペしてターミナルに貼って実行してください

```
$ git remote add origin git@github.com:<Githubのユーザネーム>/test.git
```

↑の`origin`というところはどんな文字列でもいいです

リモートリポジトリの場所を確認したい場合は`git config -l`と叩きましょう

```
$ git config -l
```

いろいろ表示されますが

```
remote.origin.url=git@github.com:e205723/test.git
```

↑こんな行があると思います、これでリモートリポジトリの場所を確認する感じです

そして念願の`git push`です

リモートリポジトリの登録をしてから、1回目の`push`は`-u origin master`を添えて実行してください

```
$ git push -u origin master
```

実行に成功すると以下の様なログが出力されます

```
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Writing objects: 100% (3/3), 224 bytes | 224.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:e205723/test.git
 * [new branch]      master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.
```

ここまでの作業で下の図まで進めることができました

<img src="/images/git/10.png" alt="10.png">

## .gitignore

`git add .`を乱用して`git status`を怠っていると`add`してはいけないファイルを`add`してしまうこともあります

こういうときに`.gitignore`ファイルを作成すると`git add .`しても特定のファイルが`add`されないようにすることができて便利です

実際に`.gitignore`ファイルを作ってみましょう

```
$ vim .gitignore
```

`.DS_Store`ファイル、`log`拡張子のファイルなどは`push`しないほうがいいですので

```
.DS_Store

*.log
```

という風に書いて保存してみましょう

そうしたら`git add .`しても`add`されないようになります

## git commit --amend

直前のコミットに間違いがあって、新たに`commit`するほどでもない間違いのとき、`commit --amend`を使いましょう


`hoge.txt`の中身を

```
hoge 1
hoge 2
```

としましょう

そして、以下のコマンドを実行します

```
$ git add hoge.txt
$ git commit -m "add hoge2"
```

このとき、`hoge.txt`の中身を

```
hoge 1
hoge 3
```

にして`commit`したかったことに気がついたとします

実際に、`hoge.txt`の中身を上記のように書き換えて、`add hoge.txt`とした後に`commit --amend`を実行してみましょう

```
$ git add hoge.txt
$ git commit --amend
```

実行すると以下の様な画面になります

<img src="/images/git/11.png" alt="11.png">

`add hoge2`を`add hoge3`にしてから保存して抜けてください

ここで

```
$ git log
```

とすると

```
commit 7272df0450d989f5403cd234c191609bb50265b5 (HEAD -> master)
Author: hoge hoge <hoo@bar> 
Date:   Sat May 15 19:29:58 2021 +0900

    add hoge3

commit c91ff94f6664388c5032a6466f64e00dac908c50 (origin/master, origin/HEAD)
Author: hoge hoge <hoo@bar>
Date:   Fri May 14 22:27:40 2021 +0900

    1st commit
```

上のようにコミットのログが出力されます

これで無駄にコミットの履歴を増やすことなく、編集したファイルをローカルの**リポジトリ**に置くことができました

## git reset

コミットのログから、任意のバージョンへ戻す方法を紹介します

まず、`hoge.txt`の中身を

```
hoge 1
hoge 2
hoge 3
```

にします、ここで

```
$ git add hoge.txt
```

とします、ここで「あー、`hoge.txt`の中身は前のままでよかったな...」ってなったら

```
$ git reset --hard HEAD
```

とすることで、**作業ディレクトリ**が前回のコミットの状態に戻って**ステージングエリア**も綺麗になります

ここで、`hoge.txt`の中身を確認してみると

```
hoge 1
hoge 3
```

となって前回のコミットの状態に戻っていることが確認できます

前々回のコミットに戻すときは

```
$ git reset --hard HEAD^
```

と打ちます、代わりに`git log`から取得したidをコピペしていくのでもありです

↓`git log`を実行した結果

```
commit 7272df0450d989f5403cd234c191609bb50265b5 (HEAD -> master)
Author: hoge hoge <hoo@bar>
Date:   Sat May 15 19:29:58 2021 +0900

    add hoge3

commit c91ff94f6664388c5032a6466f64e00dac908c50 (origin/master, origin/HEAD)
Author: hoge hoge <hoo@bar>
Date:   Fri May 14 22:27:40 2021 +0900

    1st commit
```

前々回のコミットはメッセージが「1st commit」です

そのコミットのidは例では

`c91ff94f6664388c5032a6466f64e00dac908c50`

となっていますが先頭の数桁(最低7桁以上)から一意に定まればいいでの先頭7桁だけ取って`git reset --hard`に添えて実行します

```
$ git reset --hard c91ff94
```

と実行してください

ここで、`hoge.txt`を実行すると

```
hoge 1
```

となり、最初のコミットのバージョンに戻ります

また、「`git reset`コマンドでバージョン戻したけど、もともとのバージョンが実は良かった」なんてときは、`git reset`によってキャンセルされたコミットが`ORIG_HEAD`に1つだけ入っているので

```
$ git reset --hard ORIG_HEAD
```

と実行すると、バージョンが戻ります

## git branch

`branch`の概念を勉強します

ブランチというのは「分岐」や「枝分かれ」などの意味を持ち、別々のバージョンを並行して開発するときに使います

<img src="/images/git/12.png" alt="12.png">

さっそく`git branch`コマンドを実行してみましょう

```
$ git branch
```

実行すると、現在あるブランチの一覧がでます

```
* master
```

`*`がついているブランチは現在のブランチです

ブランチが`master`しかないので、`hoge`という名前の新しいブランチを作成します

```
$ git branch hoge
```

`git branch`を実行してブランチ確認した結果が↓

```
  hoge
* master
```

`*`がついている`master`が現在のブランチです

`git checkout`コマンドでブランチ`hoge`に移動してみましょう

```
$ git checkout hoge
```

`git branch`でブランチ確認した結果が↓

```
* hoge
  master
```

ここで、`touch hoge2.txt`として`ls`

```
hoge.txt   hoge2.txt
```

ここで`git add`、`git commit`を実行

```
$ git add hoge2.txt
$ git commit -m "add hoge2.txt"
```

ここで`git log`を実行します

```
commit f960b5a75e603759a2dbd9931288d778a509cea9 (HEAD -> hoge)
Author: Yoshiaki Sano <hoo@bar>
Date:   Fri May 21 01:02:41 2021 +0900

    add hoge2.txt

commit 7272df0450d989f5403cd234c191609bb50265b5 (master)
Author: Yoshiaki Sano <hoo@bar>
Date:   Sat May 15 19:29:58 2021 +0900

    add hoge3

commit c91ff94f6664388c5032a6466f64e00dac908c50 (origin/master, origin/HEAD)
Author: Yoshiaki Sano <hoo@bar>
Date:   Fri May 14 22:27:40 2021 +0900

    1st commit
```

というふうに表示されます

メッセージが`add hoge3`であるコミット(`master`ブランチ)から`hoge`ブランチへ分岐しているのでこのような結果になります

ここまでの作業で下の図まで進めることができました

<img src="/images/git/13.png" alt="13.png">

`master`ブランチで作業したくなったら

```
$ git checkout master
```

上記のコマンドでブランチを`master`に変えてください

そうすると`git log`の結果も`ls`の結果も`hoge`ブランチに分岐する前の`master`ブランチの状態に戻ります

## git merge

あるブランチで作成/編集したファイルを別のブランチにも反映させたいときに`git merge`コマンドを使います

`git checkout`コマンドで、反映対象のブランチに移動します

`hoge`にあるファイルを`master`ブランチにも存在するようにしたいので

 現在のブランチが`master`でなかったら`git checkout master`を実行します

```
$ git checkout master
```

そしたら`hoge`ブランチを対象に`git merge`コマンドを実行してみましょう

```
$ git merge hoge
```

実行すると、`hoge`にあるファイル、コミットが`master`に反映されます

`ls`を実行すると

```
hoge.txt   hoge2.txt
```

となります

`git log`でコミットログを確認してみると

```
commit f960b5a75e603759a2dbd9931288d778a509cea9 (HEAD -> master, hoge)
Author: Yoshiaki Sano <hoo@bar>
Date:   Fri May 21 01:02:41 2021 +0900

    add hoge2.txt

commit 7272df0450d989f5403cd234c191609bb50265b5
Author: Yoshiaki Sano <hoo@bar>
Date:   Sat May 15 19:29:58 2021 +0900

    add hoge3

commit c91ff94f6664388c5032a6466f64e00dac908c50 (origin/master, origin/HEAD)
Author: Yoshiaki Sano <hoo@bar>
Date:   Fri May 14 22:27:40 2021 +0900

    1st commit
```

という感じで、`hoge2.txt`が追加されたコミットがmasterにも反映されました

ここまでの作業で下の図まで進めることができました

<img src="/images/git/14_.png" alt="14.png">

また、`hoge`ブランチを二度と使わないことになって消去したくなったら`git branch -d`コマンドを使いましょう

```
$ git branch -d hoge
```

と実行すると`hoge`ブランチが消えます、下は実行結果の出力です

```
Deleted branch hoge (was f960b5a).
```

今回のケースでは、この後に`git branch`としても`master`ブランチしか表示されなくなります

### コンフリクトとその解消

`hogehoge`という名前のブランチを新しく作って、そのブランチに移動しましょう

```
$ git branch hogehoge
$ git checkout hogehoge
```

としても良いのですが

```
$ git checkout -b hogehoge
```

として一気にブランチの生成と移動を行うことができます

上のコマンドを実行したあとに`git branch`を実行した結果が↓

```
* hogehoge
  master
```

`hogehoge`ブランチで`hoge.txt`を編集しましょう

```
hoge 1
hoge 3
```

から

```
hoge 1
hoge 2
hoge 3
```

にして保存してください

そうしたら

```
$ git add hoge.txt
$ git commit -m "add hoge2"
```

として、`hogehoge`ブランチで`git add`、`git commit`してください

ここで、一旦`master`ブランチへ移動しましょう

```
$ git checkout master
```

そして`master`ブランチのhoge.txtを編集しましょう

```
hoge 1
hoge 3
```

から

```
hoge one
hoge two
hoge three
```

にして保存してください

そしたら

```
$ git add hoge.txt
$ git commit -m "change the numerical notation"
```

として、`hogehoge`ブランチで`git add`、`git commit`してください

ここまでの作業で下の図まで進めることができました

<img src="/images/git/15.png" alt="15.png">

ここから`master`ブランチに`hogehoge`ブランチを`merge`してみましょう

`master`ブランチにて

```
$ git merge hogehoge
```

とすると

```
Auto-merging hoge.txt
CONFLICT (content): Merge conflict in hoge.txt
Automatic merge failed; fix conflicts and then commit the result.
```

このように**コンフリクト**が起きたという旨のメッセージが出力されます

**コンフリクト**とは、`merge`により結合する際に、それぞれのブランチで同じファイルの重複する部分が変更されていると発生する衝突のことを意味します

ここからは**コンフリクト**の解消方法について勉強していきます

現在の状況で`git status`を実行したときの出力の中に

```
Unmerged paths:
  (use "git add <file>..." to mark resolution)
	both modified:   hoge.txt
```

というメッセージがあります

これは両方のブランチで`hoge.txt`が編集されているという意味です

コンフリクトはこの`hoge.txt`を編集することで解決することができます

```
$ vim hoge.txt
```

とすると

```
<<<<<<< HEAD
hoge one
hoge two
hoge three
=======
hoge 1
hoge 2
hoge 3
>>>>>>> hogehoge
```

`hoge.txt`の中身が↑のようになっていることが確認できます

`<<<<<<< HEAD`と`=======`の間は、`master`ブランチの`hoge.txt`の内容です

`=======`と`>>>>>>> hogehoge`の間は、`hogehoge`ブランチの`hoge.txt`の内容です

コンフリクトは`<<<<<<< HEAD`と`=======`と`>>>>>>> hogehoge`を消して、`master`ブランチの`hoge.txt`の内容と`hogehoge`ブランチの`hoge.txt`の内容を参考にして、`hoge.txt`の中身を修正することで解消されます

`hoge.txt`の中身を`master`ブランチの1,3行目、`hogehoge`ブランチの2行目を参考にして以下のように修正してください

```
hoge one
hoge 2
hoge three
```

修正したら保存して

```
$ git add hoge.txt
$ git commit -m "fix conflict"
```

として**コンフリクト**を解消した`hoge.txt`を`git add`、`git commit`してください

これで**コンフリクト**が解消されました


