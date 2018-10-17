# PowerShell便利コマンド一覧

## 目次

<!-- - [コマンドのヘルプを調べる](#コマンドのヘルプを調べる)
- [ファイルやフォルダーを操作する](#ファイルやフォルダーを操作する)
- [ファイル検索(再帰的)](#ファイル検索(再帰的))
- [ファイル内検索(再帰的) find/grep](#ファイル内検索(再帰的)_find/grep)
- [特定のコマンドのエイリアスを調べる](#特定のコマンドのエイリアスを調べる)
- [エイリアスに割り当てられているコマンドを調べる](#エイリアスに割り当てられているコマンドを調べる)
- [CUIでのリモートアクセス(WinRM)](#CUIでのリモートアクセス(WinRM))
- [CUIでのリモートアクセス(SSH)](#CUIでのリモートアクセス(SSH)) -->

1. [コマンドのヘルプを調べる](#help)
1. [ファイルやフォルダーを操作する](#control)
1. [ファイル検索(再帰的)](#find)
1. [ファイル内検索(再帰的) find/grep](#grep)
1. [特定のコマンドのエイリアスを調べる](#select)
1. [エイリアスに割り当てられているコマンドを調べる](#gal)
1. [CUIでのリモートアクセス(WinRM)](#winrm)
1. [CUIでのリモートアクセス(SSH)](#ssh)

<!-- ## コマンドのヘルプを調べる -->

## <h2 id="help">コマンドのヘルプを調べる</h2>

- ローカルにヘルプが存在しないため事前にダウンロード必須  
 [PowerShellのヘルプファイルをローカルにインストールする](http://bakemoji.hatenablog.jp/entry/2014/01/01/195137)

```powershell
Get-Help <Command>
```

<!-- ## ファイルやフォルダーを操作する -->

## <h2 id="control">ファイルやフォルダーを操作する</h2>

- 参考  
 [PowerShellでファイルやフォルダーを操作する](http://www.atmarkit.co.jp/ait/articles/0809/05/news144.html)

PowerShellにおけるファイル操作にかかわるコマンドレット

|コマンドレット  |主なエイリアス  |概要  |
|--  |--  |--  |
|New-Item  |ni  |新規のファイル／フォルダーを作成  |
|Remove-Item  |rm／rmdir／del  |既存のファイル／フォルダーを削除  |
|Copy-Item  |copy／cp  |既存のファイル／フォルダーをコピー  |
|Move-Item  |move／mv  |既存のファイル／フォルダーを移動  |

<!-- ## ファイル検索(再帰的) -->

## <h2 id="find">ファイル検索(再帰的)</h2>

フォルダ以下に存在する拡張子.sqlのファイルをフルパスでリストアップします。

拡張子がsqlのファイルをリストアップ

```powershell
ls -Recurse -Include *.sql  | %{$_.fullname}
```

<!-- ## ファイル内検索(再帰的)_find/grep -->

##  <h2 id="grep">ファイル内検索(再帰的) find/grep</h2>

### 1.エイリアスを割り当てる

Get-ChildItemはlsという別名(Alias)を持っています。  
Select-Stringも長いので、grepで使えるようにしてしまいます。

```powershell
Set-Alias grep Select-String
```

### 2.以下のようなコマンドを実行

```powershell
ls -r -include *.cpp,*.hpp,*.h | grep '#include <boost'
```

<!-- ## 特定のコマンドのエイリアスを調べる -->

## <h2 id="select">特定のコマンドのエイリアスを調べる</h2>

```powershell
Get-Alias | Out-String -Stream |Select-String "Select"

#もしくは以下のコマンド
gal | Out-String -Stream |sls "Select"
```

実行結果

```powershell
Alias           select -> Select-Object
Alias           sls -> Select-String
```

<!-- ## エイリアスに割り当てられているコマンドを調べる -->

## <h2 id="gal">エイリアスに割り当てられているコマンドを調べる</h2>

```powershell
Get-Alias ls

#もしくは以下のコマンド
gal ls
```

実行結果

```powershell
CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Alias           ls -> Get-ChildItem
```

<!-- ## CUIでのリモートアクセス(WinRM) -->

## <h2 id="winrm">CUIでのリモートアクセス(WinRM)</h2>

PowerShellでWIndowsServerに対してリモートでアクセスする。

- アクセスできる権限はリモートデスクトップの場合と同様。
- 詳細は以下を参照。  
 [WinRM(Windows リモート管理 )　～　windows](http://memo.eightban.com/windows/winrm)
- 参考  
 [別端末(Windows)のプログラムを標準機能でリモート起動する方法まとめ](https://qiita.com/0829/items/5518256b348521ac358c)  
 [PowerShell 6 へリモート接続する(Windows 編)](http://www.vwnet.jp/windows/PowerShell/2018020501/ConnectRemotePS6.htm)

### 1.別なコンピュータへのリモート接続(WinRM)

```powershell
Enter-PSSession <hostname>  -Credential <username>

#もしくは以下のコマンド
etsn <hostname>  -Credential <username>
```

### 2.コマンドをリモートで投げる(WinRM)

```powershell
Invoke-Command <hostname> -Credential <username> {<Command>}

#または以下のコマンド
icm <hostname> -Credential <username> {<Command>}

```

### 3.ファイル転送(WinRM)

※実行クライアントはPowerShell 5.0以上

- 参考  
 [PowerShell の Copy-Item コマンドレットを使用して WinRM でファイル転送する方法](https://blog.ipswitch.com/jp/use-powershell-copy-item-cmdlet-transfer-files-winrm)  
 [PowerShell で リモートPC に フォルダ または ファイル をコピーする](http://kameyatakefumi.hatenablog.com/entry/2016/12/20/154900)

```powershell
Copy-Item –Path <FilePath> –Destination '<RemoteFilePath(Absolute)>' –ToSession (New-PSSession –ComputerName <HostName> -Credential <UserName>)

#もしくは以下を実行
cp –Path <FilePath> –Destination '<RemoteFilePath(Absolute)>' –ToSession (nsn –ComputerName <HostName> -Credential <UserName>)
```

<!-- ## CUIでのリモートアクセス(SSH) -->

## <h2 id="ssh">CUIでのリモートアクセス(SSH)</h2>

- Windows10 1803以上であれば標準インストール済み。  
 それ以外は以下からインストール。  
`https://github.com/PowerShell/Win32-OpenSSH/releases`

- クライアント/サーバーへのインストール方法(Windows10以外)  
 ※PowerShell6(PS6)のインストールは不要。  
PS6ではないのでEnter-PSSession等のコマンドは利用できないので注意。

  - [PowerShell 6 へリモート接続する(クロスプラットフォーム/パスワード認証編)](http://www.vwnet.jp/Windows/PowerShell/2018031701/PsRemoteOverSSH.htm)
  - [PowerShell 6 へリモート接続する(クロスプラットフォーム/公開鍵認証編)](http://www.vwnet.jp/Windows/PowerShell/2018032101/PsRemoteOverSSHwKey.htm)
  - [WindowsでOpenSSHを使ってSSHサーバーの起動と接続をする方法~PowerShell編](https://qiita.com/syui/items/9cd191a3395a6ebf48c1)

### 1.リモートでアクセスする(SSH)

- 参考  
 [ssh - リモートマシンにSSHでログイン - Linuxコマンド](https://webkaru.net/linux/ssh-command/)

```powershell
ssh <username>@<hostname>
```

### 2.コマンドをリモートで投げる(SSH)

- 参考  
 [【 ssh 】 SSHでリモート・マシンのコマンドを実行する](https://tech.nikkeibp.co.jp/it/article/COLUMN/20060227/230889/)

```powershell
ssh [-l user] [-i file] [-p port] [-x] host [command [arg...]]
```

|引数  |解説  |
|--  |-- |
|-l user  |ログインに使用するユーザー名を指定する  |
|-i file  |公開かぎファイルを指定する。初期設定は~/.ssh/identity  |
|-p port |接続するポートを設定する  |
|-X  |Xのポート・フォワーディングを有効にする。リモート・マシンのXアプリケーションを実行できるようになる  |
|-x  |Xのポート・フォワーディングを無効にする  |
|host  |接続するホストを指定する。user@hostのように指定することで，-lオプションと同様な効果を得ることができる  |
|command [arg...]  |リモートで実行するコマンドを指定する  |

### 3.ファイル転送(SSH)

- 参考  
 [【 scp 】コマンド――リモートマシンとの間でファイルをコピーする](http://www.atmarkit.co.jp/ait/articles/1701/27/news009.html)

```powershell
scp [オプション] [ログイン名@ホスト名]パス [ログイン名@ホスト名]パス
```

- 主なオプション

|オプション  |意味  |
|-- |-- |
|-B  |バッチモードで動作する（パスワード入力などを求めない）  |
|-l リミット  |使用帯域をKbit／秒単位で指定する  |
|-p  |コピー元のタイムスタンプやパーミッションを保持する  |
|-P ポート番号  |接続に使用するポート番号を指定する  |
|-i IDファイル  |接続に使用する公開鍵ファイルを指定する  |
|-C  |全ての通信を圧縮する  |
|-c 暗号化方法  |通信を暗号化する方法を指定する（「3des」「blowfish」「des」が指定可能）  |
|-1  |SSHv1（SSHプロトコルバージョン1）だけを使用する  |
|-2  |SSHv2（SSHプロトコルバージョン2）だけを使用する  |
|-4  |IPv4だけを使用する  |
|-6  |IPv6だけを使用する  |
|-F 設定ファイル  |SSHの設定ファイルを指定する  |
|-o 設定パラメータ  |SSHの設定パラメータを指定する（設定ファイルに書かれた内容より優先される）  |
|-q  |エラーメッセージや診断メッセージを表示しない（quiet mode）  |
|-v  |デバッグメッセージを表示する（verbose mode）  |
