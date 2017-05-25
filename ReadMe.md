# このソフトウェアについて

アカウントのupdateサブコマンドのうちSSH鍵の更新を実装した。

2FA関連は当分実装しない予定。

# 変更箇所

* `./cui/register/command/Updater.py`にSSH鍵更新処理を追記した
* `.//cui/register/github/api/v3/users/SshKeys.py`にSSH鍵削除処理を追記した
    
## バグ修正

## 関係ないSSH configファイルのホスト名定義が削除されてしまう

* `cui/register/SshConfigurator.py`

after
```python    
if nowHost != targetHost:
    nowHost = None
    afterText += line + '\n'
```
before
```python
if nowHost != targetHost:
    nowHost = None
    continue
```

## Accountsテーブル変更に合わせて修正した

after
```python
def __CreateRecordAccount(self, args, mailaddress):
    return dict(
        Username=args.username,
        MailAddress=mailaddress,
        Password=args.password,
        CreatedAt="1970-01-01T00:00:00Z",
        UpdatedAt="1970-01-01T00:00:00Z"
    )
```
before
```python
def __CreateRecordAccount(self, args, mailaddress):
    return dict(
        Username=args.username,
        MailAddress=mailaddress,
        Password=args.password,
        CreateAt="1970-01-01T00:00:00Z"
    )
```

## updateサブコマンド引数の`-p`が必須になっていた

* `./GitHubUserRegister.py`

after
```python
parser_update.add_argument('-p', '--password', '--pass')
```
before
```python
parser_update.add_argument('-p', '--password', '--pass', required=True)
```

# 前回まで

* https://github.com/ytyaru/GitHub.Upload.UserRegister.Update.Accounts.Table.201704080727
* https://github.com/ytyaru/GitHub.Upload.UserRegister.Abolition.Tsv.201704071436
* https://github.com/ytyaru/GitHub.Upload.UserRegister.Delete.Token.SshKey.201704070801
* https://github.com/ytyaru/GitHub.Upload.Connect.Token.SshKey.201704052053
* https://github.com/ytyaru/GitHub.Upload.Create.Token1.201704052033
* https://github.com/ytyaru/GitHub.Upload.Arrangement.AccountCommand.201704051735
* https://github.com/ytyaru/GitHub.Upload.Read.SshConfig.201704051338
* https://github.com/ytyaru/GitHub.Upload.GetMailAddressForAPI.201704041114
* https://github.com/ytyaru/GitHub.Upload.SSH.Check.201704040709
* https://github.com/ytyaru/GitHub.Upload.SSH.key.gen.201704031547
* https://github.com/ytyaru/GitHub.DB.Accounts.Create.Table.SshKeys.201704031424
* https://github.com/ytyaru/GitHub.Upload.UserRegister.Insert.Token.201704031122
* https://github.com/ytyaru/GitHub.Upload.UserRegister.CUI.201703311858
* https://github.com/ytyaru/GitHub.Upload.CUI.Account.SubCommand.201703302040
* https://github.com/ytyaru/GitHub.Upload.Headers.License.201703301853
* https://github.com/ytyaru/GitHub.Upload.Http.Response.201703301533
* https://github.com/ytyaru/GitHub.Upload.Response.201703291452
* https://github.com/ytyaru/GitHub.Upload.Delete.CommentAndFile.201703281815
* https://github.com/ytyaru/GitHub.Upload.Add.CUI.Directory.201703281703
* https://github.com/ytyaru/GitHub.Upload.ByPython.Add.Database.Create.refactoring.GitHubApi.201703280740
* https://github.com/ytyaru/GitHub.Upload.ByPython.Add.Database.Create.refactoring.Pagenation.201703271311
* https://github.com/ytyaru/GitHub.Upload.ByPython.Add.Database.Create.refactoring.Response.201703270700
* https://github.com/ytyaru/GitHub.Upload.ByPython.Add.Database.Create.refactoring.Data.201703261947
* https://github.com/ytyaru/GitHub.Upload.ByPython.Add.Database.Create.Callable.refactoring.201703261352

# 開発環境

* Linux Mint 17.3 MATE 32bit
* [Python 3.4.3](https://www.python.org/downloads/release/python-343/)
* [SQLite](https://www.sqlite.org/) 3.8.2

## WebService

* [GitHub](https://github.com/)
    * [アカウント](https://github.com/join?source=header-home)
    * [AccessToken](https://github.com/settings/tokens)
    * [Two-Factor認証](https://github.com/settings/two_factor_authentication/intro)
    * [API v3](https://developer.github.com/v3/)

# 準備

## 1. GitHubアカウント登録

1. [GitHubアカウント登録ページ](https://github.com/join)にてアカウントを登録する
1. 指定したメールアドレスにGitHubからメールが届くので確認する
1. リンクをクリックしてメールアドレスを`verified`にする

GitHubアカウント登録と有効なメールアドレスの登録が完了した。

## 2. SSH設定

SSH設定を自動で新規作成したいなら不要。

自分で設定したいなら、例えばGitHubユーザ名`user1`の場合、以下のようにする。

### 秘密鍵ファイルを作成する

```sh
ssh-keygen -t rsa -b 4096 -P "" -C "メールアドレス" -f "~/.ssh/rsa_4096_user1"
```

* 秘密鍵ファイル`~/.ssh/rsa_4096_user1`ができる
* 公開鍵ファイル`~/.ssh/rsa_4096_user1.pub`ができる

### configファイル

```
Host github.com.user1
  User git
  Port 22
  HostName github.com
  IdentityFile /home/mint/.ssh/github/rsa_4096_user1
  TCPKeepAlive yes
  IdentitiesOnly yes
```

## 3. DB登録

```sh
$ python3 GitHubUserRegister.py insert -u user1 -p pass -s github.com.user1 -s github.com.user1
```

SSH設定を自動で新規作成したいなら`-s github.com.user1`は不要。

# 実行

SSH鍵を作成する。
```sh
ssh-keygen -t rsa -b 2048 -P "" -C "メールアドレス" -f "~/.ssh/rsa_2048_user1"
```
`~/.ssh/config`ファイルを変更する。
```
Host github.com.user1
  User git
  Port 22
  HostName github.com
  IdentityFile /home/mint/.ssh/github/rsa_2048_user1
  TCPKeepAlive yes
  IdentitiesOnly yes
```

```sh
$ python3 GitHubUserRegister.py update -u user1 -s github.com.user1
```

SSH鍵を変更する。

# 結果

1. GitHubに登録したSSH公開鍵を削除し、新しく作成する
2. DBを更新する

# クリアした課題

* アカウント登録のupdateコマンドのうちSSH鍵更新が未実装

# 課題

* ユーザ名の更新について動作未確認
* メールアドレスの更新について動作未確認
* アカウント管理で2FA関連が未実装
    * 2FA有効アカウント未作成
    * 有効OTP生成するPythonライブラリで認証確認できていない
    * DB保存未実装
* ふだん使うTokenに権限がたくさん付与されてしまう
    * 通信傍受などでTokenが漏洩したときに危険（SSH鍵が変更されてしまうかも？）
        * `admin:public_key`の権限は初回限りにする
            * SSH鍵設定したらすぐに権限を削除する
* SSH鍵生成時に、任意の値を指定したい
    * 暗号化方式
    * 暗号化強度
* SSHのconfigを編集したい（定義順などきれいに整形したい）
* AccessTokenの`Scope`と`Grant`の文言を統一したい
* `./cui/register/command/`の`Inserter.py`と`Updater.py`に重複コード多数あるので解消したい
* `/cui/register/`の`github/api/v3`は`/web/service`のと重複する
    * 認証部分のコードが多数重複している
    * アカウント作成前なので統合できない

# ライセンス

このソフトウェアはCC0ライセンスである。

[![CC0](http://i.creativecommons.org/p/zero/1.0/88x31.png "CC0")](http://creativecommons.org/publicdomain/zero/1.0/deed.ja)

Library|License|Copyright
-------|-------|---------
[requests](http://requests-docs-ja.readthedocs.io/en/latest/)|[Apache-2.0](https://opensource.org/licenses/Apache-2.0)|[Copyright 2012 Kenneth Reitz](http://requests-docs-ja.readthedocs.io/en/latest/user/intro/#requests)
[dataset](https://dataset.readthedocs.io/en/latest/)|[MIT](https://opensource.org/licenses/MIT)|[Copyright (c) 2013, Open Knowledge Foundation, Friedrich Lindenberg, Gregor Aisch](https://github.com/pudo/dataset/blob/master/LICENSE.txt)
[bs4](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)|[MIT](https://opensource.org/licenses/MIT)|[Copyright © 1996-2011 Leonard Richardson](https://pypi.python.org/pypi/beautifulsoup4),[参考](http://tdoc.info/beautifulsoup/)
[pytz](https://github.com/newvem/pytz)|[MIT](https://opensource.org/licenses/MIT)|[Copyright (c) 2003-2005 Stuart Bishop <stuart@stuartbishop.net>](https://github.com/newvem/pytz/blob/master/LICENSE.txt)
[furl](https://github.com/gruns/furl)|[Unlicense](http://unlicense.org/)|[gruns/furl](https://github.com/gruns/furl/blob/master/LICENSE.md)
[PyYAML](https://github.com/yaml/pyyaml)|[MIT](https://opensource.org/licenses/MIT)|[Copyright (c) 2006 Kirill Simonov](https://github.com/yaml/pyyaml/blob/master/LICENSE)

