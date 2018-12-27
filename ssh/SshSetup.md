# SSHセットアップ全般

## SSHのセットアップ手順

### 前提

下記の内容はLinux同士(UNIX系)のアクセスを前提とする。  
そのため、コマンドは全てLinuxで記載している。  
PCまたはホストがWindowsの場合は、権限制御が異なるため、別途検討が必要な可能性がある。

### 概要

SSHでのアクセスを実現するには気を付ける点が複数存在する。  
下記の手順通りに実施することで、躓くポイントを回避する。  
特に以下の点に躓きやすい。  

- 鍵の生成及び管理
- 関連ファイル・ディレクトリの権限設定
- ssh-agentの設定(パスフレーズの入力省略等)

### 鍵の生成(PC側の設定)

1. 鍵の生成
    1. 次の内容を参考に鍵を生成する。  
     [Windows10におけるGit&SSHインストール手順](../Git/Windows10におけるGit&SSHインストール.md)

    2. 権限を変更する。
        ```bash
        # 権限変更
        $ chmod 700 ~/.ssh
        $ chmod 600 ~/.ssh/id_rsa
        ```

2. 設定ファイルの生成
    1. 次のコマンドを実行する。
        ```bash
        # 設定ファイル生成
        $ touch ~/.ssh/config
        ```
    2. configに次の内容を追加する。  
        ※意味は上記リンク先を参照。
        ```config:~/.ssh/config
        Include */config
        ```
    3. configの権限を変更する。
        ```bash
        # 権限変更
        $ chmod 600 ~/.ssh/config
        ```
    4. 任意のディレクトリを作成し、設定ファイルを作成する。
        ```bash
        $ cd ~/.ssh
        $ mkdir <directoryname>
        $ touch <directoryname>/config
        # 権限変更
        $ chmod 600 <directoryname>/config
        ```
    5. 設定ファイル(&lt;directoryname&gt;/config)に次のような内容を追加する。
        ```config
        Host <hostname>
            HostName <hostname>
            Compression yes
            User <username>
            IdentityFile ~/.ssh/id_rsa
            IdentitiesOnly yes
            AddKeysToAgent yes
        ```
3. SSH鍵の追加
    1. ssh-agentを起動する。
        ```bash
        # ssh-agentを起動していないとssh-addが実行できない。
        $ ssh-agent bash
        # または(Windowsは以下)
        $ eval `ssh-agent`
        ```
    2. SSH秘密鍵を追加する。
        ```bash
        # 毎回SSHアクセス時にパスフレーズを入力する手間を削減する。
        $ ssh-add ~/.ssh/id_rsa
        ```
    3. 参考  
    [【SSH】ssh-agentの使い方を整理する](https://qiita.com/Yarimizu14/items/6a4bab703d67ea766ddc)

### ホスト側の設定

1. サーバーへの公開鍵追加。
    1. SSH公開鍵をサーバーに登録する。
        ```bash
        # 公開鍵をサーバーに転送
        $ scp ~/.ssh/id_rsa.pub <username>@<servername>:'~/.ssh/'
        # サーバーにログイン
        $ ssh <username>@<servername>
        # 公開鍵を追加
        $ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
        # 権限変更
        $ chmod 600 authorized_keys
        # 所有者変更(ログインユーザが所有者ではない場合に実行)
        $ chown <username>:<username> authorized_keys
        ```
    2. ".ssh"ディレクトリの権限を変更する。
        ```bash
        # 権限変更
        $ chmod 700 ~/.ssh
        # 所有者変更(ログインユーザが所有者ではない場合に実行)
        $ chown <username>:<username> ~/.ssh
        ```
    3. 参考
        - [sshで公開鍵認証を使ってアクセスする](https://qiita.com/mountcedar/items/43157ff1225c56500655)

### ssh-agentの設定(PC側の設定)

1. expectをインストール
   1. 次のコマンドを実行。  
        対話入力を自動化するコマンド(expect)をインストール。
        ```bash
        # RHEL or CentOS
        yum install expect
        # Debian or Ubuntu
        apt-get install expect
        ```

2. ssh-agentの起動とパスフレーズの入力を自動化
    1. ~/.bashrcに以下の内容を追加する。  
        これにより、ssh-agentの起動とパスフレーズの入力を自動化する。  
        ```bash
        # ファイル名：~/.bashrc
        SSH_AGENT_FILE=$HOME/.ssh-agent
        # 秘密鍵ファイル
        KEY_FILENAME='id_rsa'
        # パスフレーズ
        PASSPHRASE='XXXXXXXXXX'

        test -f $SSH_AGENT_FILE && source $SSH_AGENT_FILE
        if ! ssh-add -l > /dev/null 2>&1; then
          ssh-agent > $SSH_AGENT_FILE
          source $SSH_AGENT_FILE
          expect -c "
          set timeout -1
          spawn ssh-add $HOME/.ssh/$KEY_FILENAME
          expect {
              \"Enter passphrase for\" {
                  send \"$PASSPHRASE\r\"
              }
          }
          expect {
              \"denied\" { exit 1 }
              eof { exit 0 }
          }
          "
        fi
        ```
    2. 参考
        - [ssh-agent と expect で ssh のパスフレーズ入力を自動化](https://qiita.com/abpla/items/52e18c393d4cf2875384)
        - [Windows 10のWindows Subsystem for Linux（WSL）を日常的に活用する](https://www.clear-code.com/blog/2017/11/8.html)
