# Windows10におけるGit&SSHインストール手順

## 目次

- [環境](#environment)
- [概要](#outline)
- [1. Git for Windowsのインストール及び設定](#gfw)
- [2. SSH鍵をIncludeを利用して管理しGitと連携する](#sshandgit)
- [3. SourceTreeからPuTTYおよびOpenSSHを利用してホスティングサービスと連携する](#sourcetree)
- [4. 複数のアカウントをGitホスティングサービスと連携する(OpenSSH)](#multiple)

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
    - 値：C:\Windows\System32\OpenSSH\ssh.exe  
        ※Win32 OpenSSHの場合は任意のインストール先を指定

## <h2 id="sshandgit">2. SSH鍵をIncludeを利用して管理しGitと連携する</h2>

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
        -Cと-fの内容は任意で修正。
        ```powershell
        ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f bitbucket/id_rsa
        ```

    2. パスフレーズの入力を求められるので、入力  
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
        User git
        IdentityFile C:\Users\{UserName}\.ssh\bitbucket\id_rsa
        IdentitiesOnly yes
        AddKeysToAgent yes
    ```

    - 参考  
    [お前らのSSH Keysの作り方は間違っている](https://qiita.com/suthio/items/2760e4cff0e185fe2db9)

4. Gitホスティングサービス用の設定を".ssh/config"にコピーする。  
    上記までの手順により、gitコマンド(及びGitBash)以外からのSSHは全て正常に動作するようになる。  
    しかし、gitコマンドからは、各プロジェクト毎のconfigファイルを読み込むことができない不具合が発生する。  
    (Includeが有効にならない)  

    そこで、下記の手順によりgitコマンドから参照されるSSH設定を読み込ませる。

    1. Gitホスティングサービスとの接続に利用するconfigの内容を".ssh/config"に追加する。
        ```powershell
        #Bitbucket
        Host bitbucket.org
            HostName bitbucket.org
            Compression yes
            User git
            IdentityFile C:\Users\{UserName}\.ssh\bitbucket\id_rsa
            IdentitiesOnly yes
            AddKeysToAgent yes
        #GitHub
        Host github.com
            HostName github.com
            Compression yes
            User git
            IdentityFile C:\Users\{UserName}\.ssh\github\id_rsa
            IdentitiesOnly yes
            AddKeysToAgent yes
        ```

    2. Windowsサービス"ssh-agent"を再起動する。

5. 公開鍵をGitホスティングサービスに登録する。  
    - 登録方法はリンク先を参考にする。  
 [[Windows] Visual Studio CodeでGithub・Gitlab・Bitbucketそれぞれにssh接続する](https://qiita.com/MegaBlackLabel/items/e825babfdc1b7fffec96)

## <h2 id="sourcetree">3. SourceTreeからPuTTYおよびOpenSSHを利用してホスティングサービスと連携する</h2>

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

## <h2 id="multiple">4.  複数のアカウントをGitホスティングサービスと連携する(OpenSSH)</h2>

- 複数のアカウントを利用する際の課題  
  - 会社用と個人用、会社の管理者用と一般用等、同一のGitホスティングサービス内で複数のアカウントを利用したい場合がある。  
  HTTPS接続の場合、上記の通りリモートリポジトリにアクセスする度に、IDとパスワードを求められてしまう。
  - そこで.ssh/configと.git/configの設定を書き換えてアクセスできるようにする。  
    なお、SourceTreeを介してPuTTYを利用する場合は、下記のような複雑な手順は不要(名前/メールアドレスは変更不可)。

1. .ssh/configを以下のように設定する  
下記の例はBitbucketだが、GitHubも同じ手順で可能。  
※事前に複数の鍵を生成していることが前提。

    ```powershell
    #会社用
    Host bitbucket.org
    HostName    bitbucket.org
    Port    22
    User    git
    IdentityFile    C:\Users\{UserName}\.ssh\bitbucket\id_rsa
    IdentitiesOnly yes
    AddKeysToAgent yes

    #個人用
    Host private.bitbucket.org
    HostName    bitbucket.org
    Port    22
    User    git
    IdentityFile    C:\Users\{UserName}\.ssh\bitbucket\id_private_rsa
    IdentitiesOnly yes
    AddKeysToAgent yes
    ```

2. 各リポジトリのリモート設定を".git/config"から書き換える(個人用の場合)。

    ```powershell
    >  git config -e

    [remote "origin"]
    url = git@private.bitbucket.org:piruty/bucket.git # <- この行を修正
    fetch = +refs/heads/*:refs/remotes/origin/*
    ```
3. Gitで使用する名前とメールアドレスを変更する(必要な場合のみ)。

    普通は以下のような設定により、名前とメールアドレスが設定される。
    ```powershell
    > git config --global user.name "handle name"
    > git config --global user.email "handle.name@community.sample.com"
    ```

    2.までの設定を行っても名前とメールアドレスは"--global"の設定が有効になっている。  
    そこで、以下のように、各リポジトリに対して"--local"の設定を行う。

    ```powershell
    > git config --local user.name "real name"
    > git config --local user.email "real.name@company.sample.com"
    ```

- 参考  
 [同一端末で、複数のGitHubアカウントを使い分ける方法](https://github.com/youkinjoh/TrainingWebSocket/wiki/%E5%90%8C%E4%B8%80%E7%AB%AF%E6%9C%AB%E3%81%A7%E3%80%81%E8%A4%87%E6%95%B0%E3%81%AEGitHub%E3%82%A2%E3%82%AB%E3%82%A6%E3%83%B3%E3%83%88%E3%82%92%E4%BD%BF%E3%81%84%E5%88%86%E3%81%91%E3%82%8B%E6%96%B9%E6%B3%95)  
[[git] Bitbucketの複数アカウントを使い分ける](https://piruty.com/?p=534)
