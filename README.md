# 模範解答

基本的な方針は、`checker.pl` が NG と判定するコミットとその修正コミットを統合することです。

まず `checker.pl` を取り出します。

```sh-session
$ git checkout master
$ git checkout origin/checker checker.pl
```

次に、この `checker.pl` がどのコミットで :imp: するのか確かめます。

なお、`checker.pl` には標準入力から CSV を渡す必要があるので、`git show` コマンドを使います。

```sh-session
$ git show <リビジョン>
```

この `git show` コマンドは、後に指定されたリビジョンで指定されたコミットやファイルなどを表示するコマンドです。
そして、あるコミット時点でのファイルを指定するリビジョンの形式は `<コミットのSHA1>:<ファイル名>` です。
試しに、`361d729` における `users.csv` の内容を確認してみましょう:

```sh-session
$ git show 361d729:users.csv
214,9278281f-7764-4f31-9d17-feb49e23f7aa,吉田 結衣,Sonia57@yahoo.com,Ep6cTpqsr1NPvkt,07-1729-7655
379,72869cdb-f0a5-41a5-b081-400f52d05471,佐藤 大翔,Noemy_Cassin86@gmail.com,s9uDi38pqjEDMgY,0052-63-7913
1,b2c50600-eed8-44bc-bb90-e4c4a1d05689,鈴木 大翔,Sydnee_Lebsack@hotmail.com,wnc3rN7tyMrLf40,0148-17-1732
...
``` 

うまく表示されました。
次に、`git show` コマンドを使って `checker.pl` を実行してみます。

```sh-session
$ git show ${commitish}:users.csv | ./checker.pl && echo OK || echo NG
OK
``` 

うまく判定できていそうなことが確認できました。

これを、`master`ブランチのすべてのコミットに対して実行してみましょう。
このとき、`git rev-list` コマンドが便利です。
`git rev-list` コマンドは、指定したリビジョンまでのすべての祖先コミットの SHA1 を表示します。

試しに、`master` のすべての祖先コミットを表示してみましょう。

```sh-session
$ git rev-list master
c5516aadd35ff4f8718e0843b7f91cda6dd96d7a
093e41a8660e75b794eede185166f3eb4cda7d07
30d56727840b7f2fa5879a14e968bcfd64b7f9c1
...
``` 

このように、新しいコミットを先頭に祖先コミットが順に表示されていることがわかります。
この表示の形式は、shell script の for ループで簡単に扱うことができます。

では、`checker.pl` によるチェックを for ループと組み合わせてみましょう。
次の `$commitish` 変数には、`git rev-list` の結果の一行（= コミットの SHA1 ）が格納されます。

```sh-session
$ for commitish in $(git rev-list HEAD); do
     echo -n "$commitish: "
    git show ${commitish}:users.csv | ./checker.pl && echo OK || echo NG
  done
361d729d1b0bac518a3944169f12061a504f15ab: OK
0f2bbf00944271b5bcec2a77cbeb898c250e2d0f: OK
f93cf3b98c34f9e3f5022ecfc5bb4c8074e99a15: OK
8458758abdb7c0a2f92594b944559eb75d8d9453: OK
1c201b5f86d22ef59da1524b2c401b7a7a655c1d: OK
c77445a6614045753c76e4b32456b920affe0652: OK
cccd3171acf2b3eef3c7428ebed5b60025d16061: NG
391eb34faa0791e1fb3827770c7c554ff40b32d0: OK
95d24e45ea5a74583a8f9a3a2b1dd38cd1731180: OK
ded0224c89498901e2fcb96fc8c1872bd15b25eb: OK
250e21a6b82562284524fb2afc1d3f2826a97ff5: OK
d3138baa592ea8dbbc53cbad90ff4460f4618b7b: OK
e66e018713e3ca001aa1186433137656838bd43b: OK
7581cb00ca96f76d3854a5af5c9f3d4fe7ec9531: OK
d30ab470dfb5f273f691c4e8499393979664b0e7: OK
b4aae055c95c65f90abe7b38052954ebcc2b1402: OK
23cec888c8ac0eb4793abfc9a1105795f32fc02c: OK
fa9edad3547b825a6de213906ace375c2fd6da92: OK
a9e67aca55590c456d29d4787eb424b77eef48d3: OK
70eab654f95068e60a9e909e87c29aeb96dca8d4: OK
5e6a1a8c124dc2005132d8d7dd41ef86d4a16f50: OK
e5197fddd44b9b8e1a95a01fce76b0018b2fe9e6: OK
e4adab4a7154e51502e64a1f68817fa48f66b9d7: OK
f7ea1267d74eb41783d3fec5f4a8743f913a13c7: OK
038f1ac25c9e07ead6b6542a0f2d76826448c07a: OK
e6f3328ccb9c164591db96835e75064ffbf2302a: NG
9155532bc27b4709671a2d62d2aab4f5288999b8: OK
2f86d6ea9ccd24fca822666ca9c8cd2c0992e2ec: OK
0d6a61609b71a7ff2c2eb5a08cefaeb3bef8c095: OK
bef23c8bd6e77fd30b36dff15929bbc4bc88fbaf: fatal: Path 'users.csv' exists on disk, but not in 'bef23c8bd6e77fd30b36dff15929bbc4bc88fbaf'.
NG
56c22779ac4455f7868af7628275d0007cf62b8b: OK
854f6190166713659fe383917973f5f3f3fb47a5: OK
09729bcc26161deabb745e2521e5708ebe01ece1: OK
dab6588f90661089b7d8173247d85c899ce90fc6: OK
94f180166c778eddae0047e48c9a3342df956f40: OK
a934c7c0035cdaf831bd7e1cbde602496b479e2c: NG
5711d2a1d8d7a999d3c066c61827e66ffc80fd57: OK
6278af9119567894a65ee3cdca51de498fea6d5d: NG
e4e4e1aaf18d3c5d50b9871855645b3a49cbd3a2: OK
578ac5bbcab385b1465e736962621e85ccfc3a98: OK
d08f34d753b7eb7511a773fc5624f0ab2d9e71d5: NG
6ddea52001bbf65b287e33eb60b30708214d66c1: OK
5c49a2cf1c23f672ba149463c3ede91ee4bc1115: OK
c3da1ab17f085f5a69c40ea359701639d6ef7ea7: OK
```

下の6つのコミットに問題があることがわかりました:

- `cccd3171acf2b3eef3c7428ebed5b60025d16061`
- `e6f3328ccb9c164591db96835e75064ffbf2302a`
- `bef23c8bd6e77fd30b36dff15929bbc4bc88fbaf`
- `a934c7c0035cdaf831bd7e1cbde602496b479e2c`
- `6278af9119567894a65ee3cdca51de498fea6d5d`
- `d08f34d753b7eb7511a773fc5624f0ab2d9e71d5`

これをメモしておきましょう。

また、これらの直後のコミットをみると、OK に戻っていることがわかります。
つまり、直後のコミットが修正コミットであるということです。

さて、この問題の正当条件は以下の2点でした。

- どのコミットに checkout しても `./checker.pl` が失敗しないこと
- コミットに過不足がないこと

この条件を満たすには、問題のあるコミットとその直後の修正コミットを1つのコミットにまとめられればよいですね。

今回のように複数のコミットを前後のコミットと結合する際には、対話的（interactive）な rebase が便利です。
対話的な rebase は下のように実行できます。

```sh-session
$ git rebase --interactive <変更したい最初のコミットの SHA1>
```

または、`--interactive` の省略形である `-i` でもよいでしょう:

```sh-session
$ git rebase -i <変更したい最初のコミットの SHA1>
```

対話的 rebase が実行されると、次のような内容でエディタが開かれます。

```
pick 5c49a2c "田中"さんからの要望対応
pick 6ddea52 "斎藤"さんからの要望対応
pick d08f34d "松本"さんからの要望対応
...
```

この `pick` というキーワードは、rebase 時のコマンドを表しています。
なお、`pick` は「何も変更を加えずに歴史に残す」というコマンドです。
`pick` の他には、次のようなコマンドがあります（ファイルの末尾にコメントで記述されています）:

| コマンド名 | 省略形 |                                                                  |
|:-----------|:-------|:-----------------------------------------------------------------|
| `pick`     | `p`    | コミットをそのまま歴史に残す                                     |
| `reword`   | `r`    | コミットを歴史に残すが、コミットメッセージを修正する             |
| `edit`     | `e`    | コミットを歴史に残すが、差分を修正する                           |
| `squash`   | `s`    | コミットを前のコミットに統合して、コミットメッセージを修正する   |
| `fixup`    | `f`    | コミットを前のコミットに統合する（このコミットメッセージは消える |
| `exec`     | `x`    | シェルでコマンドを実行する                                       |

さて、この中に `fixup` という、ちょうどよいコマンドがあることに気がつきますね。
これを使うことにしましょう。

では、実際に rebase する手順をみていきます。

今回は、歴史がそこまで深くないので、最も古いコミットから rebase してしまいましょう。
最も古いコミットから rebase したい場合には、`git rev-list --reverse | head -n 1` を併用すると便利です。

`git rev-list` は、あるコミットからのすべての祖先のコミットの SHA1 を表示するコマンドです。
これに `--reverse` をつけると、最も古いコミットの SHA1 が先頭にきます。
そして、`head -n 1` で先頭のひとつだけを取り出せば、最も古いコミットの SHA1 が手に入るというわけです！

このコマンドの実行結果をコピー＆ペーストでもよいのですが、`$(command)` や `` `command` `` といったコマンド置換をつかうとコピー＆ペーストの手間が省けます:

```sh-session
$ git rebase -i $(git rev-list --reverse HEAD | head -n 1)
```

さて、開かれたエディタを操作して、前の手順でメモした6つのコミットの後のコミットに fixup フラグをつけます。

```
pick 5c49a2c "田中"さんからの要望対応
pick 6ddea52 "斎藤"さんからの要望対応
pick d08f34d "松本"さんからの要望対応
fixup 578ac5b おっと、うっかりミス 😜
pick e4e4e1a "渡辺"さんからの要望対応
pick 6278af9 "高橋"さんからの要望対応
fixup 5711d2a 人間って、ミスをする生き物ですよね 😉
pick a934c7c "鈴木"さんからの要望対応
fixup 94f1801 ミスだってしますよ。だって人間だもの 😤
pick dab6588 "中村"さんからの要望対応
pick 09729bc "斎藤"さんからの要望対応
pick 854f619 "山本"さんからの要望対応
pick 56c2277 "井上"さんからの要望対応
pick bef23c8 "マイケル"さんからの要望対応
fixup 0d6a616 ファイルを消すな 💢💢💢💢💢
pick 2f86d6e "マイケル"さんからの要望対応
pick 9155532 "小林"さんからの要望対応
pick e6f3328 "佐藤"さんからの要望対応
fixup 038f1ac 給料上げてくれたらミスが減るとおもいます 😇
pick f7ea126 "山田"さんからの要望対応
pick e4adab4 "小林"さんからの要望対応
pick e5197fd "井上"さんからの要望対応
pick 5e6a1a8 "木村"さんからの要望対応
pick 70eab65 "小林"さんからの要望対応
pick a9e67ac "中村"さんからの要望対応
pick fa9edad "清水"さんからの要望対応
pick 23cec88 "加藤"さんからの要望対応
pick b4aae05 "伊藤"さんからの要望対応
pick d30ab47 "佐藤"さんからの要望対応
pick 7581cb0 "山田"さんからの要望対応
pick e66e018 "鈴木"さんからの要望対応
pick d3138ba "林"さんからの要望対応
pick 250e21a "吉田"さんからの要望対応
pick ded0224 "清水"さんからの要望対応
pick 95d24e4 "伊藤"さんからの要望対応
pick 391eb34 "山本"さんからの要望対応
pick cccd317 "山口"さんからの要望対応
fixup c77445a 給料上げたらミス減りましたね 😮
pick 1c201b5 "佐藤"さんからの要望対応
pick 8458758 "吉田"さんからの要望対応
pick f93cf3b "山本"さんからの要望対応
pick 0f2bbf0 "小林"さんからの要望対応
pick 361d729 "吉田"さんからの要望対応
```

このファイルを保存して閉じると、rebase が実行されます。
すると、途中で rebase が停止します。

```
interactive rebase in progress; onto c3da1ab
Last commands done (15 commands done):
   pick bef23c8 "マイケル"さんからの要望対応
   fixup 0d6a616 ファイルを消すな 💢💢💢💢💢
Next commands to do (28 remaining commands):
   pick 2f86d6e "マイケル"さんからの要望対応
   pick 9155532 "小林"さんからの要望対応
You are currently rebasing branch 'master' on 'c3da1ab'.

No changes
You asked to amend the most recent commit, but doing so would make
it empty. You can repeat your command with --allow-empty, or you can
remove the commit entirely with "git reset HEAD^".

Could not apply 0d6a61609b71a7ff2c2eb5a08cefaeb3bef8c095... ファイルを消すな 💢💢💢💢💢
```

このメッセージによれば、コミットを統合すると差分がなくなるので、対処方法を指定してほしい、とのことです。

さて、統合したかった2つのコミットをよくみると、`0d6a616` で消されたファイルをそのまま復活させていることがわかります。
つまり、fixup によってコミットを統合しようとすると、差分が発生しないのです。

そこで、これまでのコミットログを見返すと、修正コミットの直後の `2f86d6e` に、問題となっていた `bef23c8` と同じメッセージのコミットがあることに気がつきます。
つまり、本来 `bef23c8` で加えられるべき変更が、`2f86d6e` によって加えられていると推測できます。
ということは、`bef23c8` と `0d6a616` を歴史から消すと、コミットに過不足がない（正当条件）ことになります。

そこで、以下のようにして skip 相当の操作をします。

```sh-session
$ git reset HEAD^
$ git rebase --continue
```

この後は無事に rebase が完了します。

最後に、checker.pl が :imp: するようなコミットがないことを確認しましょう。

```sh-session
$ for commitish in $(git rev-list HEAD); do
     echo -n "$commitish: "
    (git show ${commitish}:users.csv | ./checker.pl) && echo OK || echo NG
  done
c5516aadd35ff4f8718e0843b7f91cda6dd96d7a: OK
093e41a8660e75b794eede185166f3eb4cda7d07: OK
30d56727840b7f2fa5879a14e968bcfd64b7f9c1: OK
6fb310b1c5a5c1459d2424a00dfce8e7b83edaf2: OK
ecc3fce0b4c12c48cf658d6b24f641638fbf982a: OK
fec06b0552324bac7525e1f3fbaf63a54b63d6eb: OK
9080aa676511aa9ad15d49fa1451e555536c5cfb: OK
479992bdd2d7775fcdb661a49d3117caee59f445: OK
12ccba83fbbf1426d95cc57e7bb5d257b9717327: OK
83c37655cd0dbecd15de405641f08003268b0ace: OK
e17fd82bc6de6a5232efd4c6689cf037410864cf: OK
99020d49d268e96e3c0074b51e025f563b92df92: OK
ebda7e45eacd0c20915b89f4778e22097869ef4f: OK
9dcce630eab102b2b0185b17e6cbe77a3f3f176c: OK
767c1040573a3252fe376055cccffddbb9c696ba: OK
26906285af379b621741f4a97c2df13980a1378f: OK
3f233a6deb55dafec7cca6abface5c35585ef29c: OK
18b9c5b936c56074a2b1fef9511ae97253556a8c: OK
09033b431c0c961e44894294ef2b38fef17b2e95: OK
e304a9d5846f1cd2454f9a9dbbd175f21ff5cf1d: OK
0db19891e2f78a561cb480b0fdb7b702d60fc193: OK
69235199570768b09af45ef5f97695c834bafb46: OK
b3fd6574f87e8183f9152048e77af960c6f12842: OK
93b2f82c405ebaf79c5d4d2a30332be3bfc4ae1d: OK
510a11640de81b167b96cae2ce3b92ca33c0b1b6: OK
3e8e9bc9126becf56f759591ff3c522ec03834a0: OK
8d330511ca3cf411af4bfb4721242fe8ee12eefb: OK
94b4b94553e5ae06b2b91fd48254a2b3e8cafce7: OK
6153f4c5f387db02cb39c719a52685e6a3dec68d: OK
4b8113cc9c4e7c952f710b41cf99905060a5d9dd: OK
e252c3d33fa00a63c3f8300e4050a485e5aa7345: OK
deecf6288d08c5b7d6dd2b38d0439806e3eecab8: OK
a446d3638431ba9ae98032c80b0d6ff2acc8246f: OK
63b7b83ff57bdfd5798313751870746a624f9e54: OK
6ddea52001bbf65b287e33eb60b30708214d66c1: OK
5c49a2cf1c23f672ba149463c3ede91ee4bc1115: OK
c3da1ab17f085f5a69c40ea359701639d6ef7ea7: OK
```

すべて OK になったことが確認できました。

push して、解答をチェックしましょう。

```sh-session
$ git push origin master
To git@github.com:mixi-git-challenge/git-challenge-minesweeper
 ! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'git@github.com:mixi-git-challenge/git-challenge-minesweeper'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
``` 

おっと、rebase で歴史を書き換えているので、origin の master が祖先にないと怒られてしまいました。
正当条件には、origin の master を祖先にもたなければならないといった指定はありませんので、force push してしまいます。

```sh-session
$ git push -f origin master
```

これで、無事に夜も眠れるリポジトリとなりました。


# 教訓

1. テストに通らないような雑なコミットをリリースブランチに残すと、前の差分に戻すときに障害が発生するリスクが増加します
2. この状態から綺麗な歴史にしようとすると、大規模な歴史の書き換えが発生します
3. リリースブランチへの commit や merge はテストに成功することを確認してからおこないましょう

とはいっても、テストで見つけられなかった不具合がリリースブランチに混入するのはよくあることです。
そのたびに、今回のように歴史を改変するのは大変なコストです。
つまり、実際には revert による改変を伴わない歴史の巻き戻しでバグを除くことになります。

そのとき、今回の問題の状況を思い出してください。
**`revert` をした後が万全な状態とは限らないのです。**
つまり、歴史の戻し先は慎重に検討する必要があります（=夜も眠れないのは変わらない）。

夜も眠れない問題の解消は、リリースブランチに差分が入る前の入念なQA活動（テストや、不具合を防ぐ開発環境・開発体制の整備など）で目指すべきでしょう。
