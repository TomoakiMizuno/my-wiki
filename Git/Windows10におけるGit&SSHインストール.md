# Windows10におけるGit&SHHインストール手順

## 目次

- [環境](#environment)
- [概要](#outline)
- [1. Git for Windowsのインストール及び設定](#gfw)
- [2. SSHの鍵をIncludeを利用して管理してGitと連携する](#sshandgit)
- [3.SourceTreeからPuTTYおよびOpenSSHを利用してホスティングサービスと連携する](#sourcetree)

## <h2 id="environment">環境</h2>

- Windows 10 1803  
 ※標準でOpenSSHを実装したバージョン
- Git for Windows 2.19.1  
  [公式サイト](https://gitforwindows.org/)
- Visual Studio Code  1.28.1  
 [公式サイト](https://azure.microsoft.com/ja-jp/products/visual-studio-code/)
- SourceTree 3.0.6  
 [公式サイト](https://ja.atlassian.com/software/sourcetree)

## <h2 id="outline">概要</h2>

Windows 10でOpenSSHが標準実装されて間もない(2018年10月現在)ことで、  
SSHを利用したGitHubやBitbucket等Gitホスティングサービスへの連携には様々な障害がある。

1. SSH接続時の問題点  
    sshコマンドで利用する際の設定ファイルとgitコマンドで利用するssh設定ファイルが異なる。  
    SSHで接続する際に利用する設定ファイル  
    ([ユーザーのホームフォルダ]/.ssh/config) がGitのSSH接続では利用されないという欠陥が存在する。  

    上記ファイルの代わりに以下のファイルが設定ファイルとして利用される。  
    [Gitをインストールしたパス]/Git/etc/ssh/ssh_config

2. SSHクライアントの問題  
    Windows10 1803以前のWindowsではOpenSSHクライアントが存在しなかった。  
    そのため、代替手段としてPuTTYを利用することが多かった。  

    OpenSSHとPuTTYは鍵の管理方法が異なり、鍵の二重管理が発生し得る。  
    例えば、VSCodeではOpenSSHが利用されるが、SourceTreeではPuTTYが標準で利用される。  
    鍵の管理が異なるため、両方を同時に利用するには2種類の鍵を生成しなくてはならない。

    更に、Git for Windows(以下GfW)のSSHクライアントと、  
    Windows 10の標準SSHクライアント(以下Win10SSH)が異なっている問題がある。  
    gitコマンド及びGit BashからはGfWに含まれるSSHクライアントが利用され、  
    PowerShellからのSSHではWin10SSHが利用される。

- 対応策  
 SSH接続時とSSHクライアントの問題の対応策を含めたインストール方法を以下に記述する。  
 なお、VSCode/SourceTreeのインストール設定については対象外とする。

## <h2 id="gfw">1. Git for Windowsのインストール及び設定</h2>

- Git for Windowsの入手とインストール  
    1. 下記リンク先を参照してインストール。  
    <https://qiita.com/elu_jaune/items/280b4773a3a66c7956fe>  
    ※標準エディタはVSCodeに設定

    2. 環境変数を設定する  
    本設定により、Windows10標準のOpenSSH及び設定ファイルが利用されるようになる。  
     ※Windows 10 1803未満であれば、[Win32 OpenSSH](https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH)を事前にインストールする。  

        コントロールパネル -> システムとセキュリティ -> システム -> システムの詳細設定  
        -> 環境変数 -> 以下を追加  

  - システム環境変数
    - 変数：GIT_SSH
    - 値：C:\Program Files\OpenSSH\ssh.exe  
        ※Win32 OpenSSHの場合は任意のインストール先を指定

## <h2 id="sshandgit">2. SSHの鍵をIncludeを利用して管理してGitと連携する</h2>

- プロジェクト毎に鍵を管理するようにする。  
  参考  
  [サーバが多くSSH鍵の管理が大変です。どんな工夫をしていますか？](https://qiita.com/suin/items/e4a976d076134f755a9a)  
 [ssh_configのIncludeでハマった話](https://tech.innovator.jp.net/entry/2018/05/24/143654)  

1. ".ssh/config"の設定を行う。  
    configファイルに次の設定を最上位に追加する。

    ```config
    Include */config
    ```

    次のようなディレクトリ構成にする。  

    ```cmd
    ├── config ... すべてのサーバに共通した設定を書くのと、*/configをIncludeする場所
    ├── id_rsa ... 使いまわす鍵
    ├── id_rsa.pub ... 使いまわす鍵
    ├── known_hosts
    ├── github ... プロジェクトごとにディレクトリを作り
    │   ├── config ... プロジェクトごとにconfigファイルを置く
    │   ├── id_rsa
    │   └── id_rsa.pub
    ├── bitbucket
    │   ├── config
    │   ├── id_rsa
    │   └── id_rsa.pub
    ├── projhoge
    │   ├── config
    │   └── id_rsa
    ├── projfuga
    │   ├── config
    │   └── id_rsa
    ...
    ```

2. プロジェクト毎に鍵を生成する。

    1. 以下のようなコマンドを実行  
        ```powershell
        ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f bitbucket/id_rsa
        ```

    2. パスフレーズの入力を求められるので、入力  
        ※VSCodeはパスフレーズに対応していない可能性大。パスフレーズ無しでそのままEnter。
        ```powershell
        Enter passphrase (empty for no passphrase): [Type a passphrase]
        # Enter same passphrase again: [Type passphrase again]
        ```

3. 各プロジェクト毎にconfigファイルを生成する。  

    - Bitbucket設定例(bitbucket/config)  
    ```config
    Host bitbucket.org
        HostName bitbucket.org
        Compression yes
        User UserName
        IdentityFile C:\Users\t-mizuno\.ssh\bitbucket\id_rsa
        IdentitiesOnly yes
    ```

    - 参考  
    [お前らのSSH Keysの作り方は間違っている](https://qiita.com/suthio/items/2760e4cff0e185fe2db9)

4. 公開鍵をGitホスティングサービスに登録する。  
    - 登録方法はリンク先を参考にする。  
 [[Windows] Visual Studio CodeでGithub・Gitlab・Bitbucketそれぞれにssh接続する](https://qiita.com/MegaBlackLabel/items/e825babfdc1b7fffec96)

## <h2 id="sourcetree">3.SourceTreeからPuTTYおよびOpenSSHを利用してホスティングサービスと連携する</h2>

- SourceTreeでHTTPSを利用しない理由  
  - HTTPS接続の場合、SourceTree内で複数アカウントを利用すると、
  Bitbucketなどのリモートリポジトリにアクセスする度に、IDとパスワードを求められてしまう。
  - Windowsファイルサーバーへのコミット内容の反映を行いたいときに、共有フォルダに自身のホスティングサービスアカウントでアクセスすることになり、セキュリティ上問題がある。

- SSH接続時の問題点
  - SourceTreeではデフォルトでPuTTYがSSHクライアントとなっている。  
  PuTTYとOpenSSHの双方での接続方法を以下に記述する。

1. PuTTYを利用した連携方法  
 リンク先を参考に設定する。  
[SourceTree 設定からSSH設定＆リポジトリをcloneするまでの流れ](https://qiita.com/github129/items/b23a24aaa359a0f8eba7)  
 ※以下の点に気を付ける。  

    1. デフォルトでは鍵の保存時に"[ユーザーのホームフォルダ]/ssh"フォルダに保存される。
        ".ssh"ではない。  
    2. OpenSSHとは鍵の互換性がない。  
        PuTTYで生成した鍵をOpenSSHでそのまま利用することは出来ない。  
        なお、PuTTYに鍵の相互変換機能がある模様。  
        しかし、同じ公開鍵を利用することに失敗したため、OpenSSHとは違う鍵を利用することを推奨。  

2. OpenSSHを利用した連携方法  

    1. SourceTreeを起動 -> ツール -> オプション -> SSH クライアント -> "OpenSSH"に変更。  
    2. "SSH キー"にssh-keygenコマンドで生成した秘密鍵を登録する。  
        例) ".ssh/bitbucket/id_rsa"等
    3. Gitホスティングサービスに公開鍵を登録する。
        - 登録方法はリンク先を参考にする。  
 [[Windows] Visual Studio CodeでGithub・Gitlab・Bitbucketそれぞれにssh接続する](https://qiita.com/MegaBlackLabel/items/e825babfdc1b7fffec96)