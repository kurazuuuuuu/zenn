---
title: "クラウドIDE？Coderを使おう"
emoji: "💻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Github", "Coder", "Minecraft", "IDE", "Cloud"]
published: false
---
# はじめに
こんにちは、はじめましての方ははじめまして、「くらず / kurazu」と申します。なんちゃってDevOpsを自宅鯖でやってる学生エンジニアです。
さて、最近はAI駆動開発が主流になりつつあるように感じます。私個人的にこの進化し続けるAI時代において必要なのは**開発力・知識量**、それ以上に大切なのは**開発環境の統一感**だと思います。今回はそんな問題を解決するための1ピースになるツールを発見したためご紹介したいと思います。

# 本編
## クラウドIDEって？
まずはクラウドIDEって何なのかという話です。最初に例を挙げますと代表的なのは**Github Codespaces, AWS Cloud9**ではないでしょうか？見たり聞いたりしたことある方が多いと思います。特にCodespacesは簡易的な開発環境として非常に使いやすいクラウドIDEだと思います。

### > クラウドIDEのメリット
わざわざクラウドIDEを使うことのメリットについて疑問をもつ方もいるかもしれません。**「Macならオンデバイスでなんでも動くじゃん」「Dockerコンテナ使ったら楽」「WSL使え」** 

まあ確かにその通りです。クラウドだと 
**「コストがかかる」「インターネット接続が必要」「初回の環境構築が面倒」**
といった問題があります。しかし、それらを覆す最大のメリットがあります。

それは...

<br>
#### \ \ 環境を完全に統一できる / / 
<br>

ローカルの環境のDocker Engine上で開発コンテナを動かしてアタッチして作業みたいな場面があると思いますが、パターンによってはそもそもDocker Engineが動かない、動きが違うなんてこともあるかもしれません。（エミュレーション環境が違えば発生することがあります（ARM, AMD64, Appleの仮想化など））

![docker-emulation](/images/coder/CleanShot%202025-12-25%20at%2000.03.55.png)

しかし、クラウドIDEであれば稼働しているホスト環境は全員統一であり、決まったテンプレートに従い環境が構築されるのでチームやプロジェクト単位で共同開発を行う際に完全に統一された開発環境で開発することができます。素晴らしいですね。

### > クラウドIDEのデメリット
もちろん完全にメリットだらけなわけではないです。クラウドIDEは大体テンプレートや初期状態を決めるためのコンテナを作成する必要があります。これには結構な時間・コストが必要でDockerに関する知識が必要です。しかし、1度クラウド環境を構築してしまえば今後長期的に安定した開発環境を何度も構築・破壊ができるため、メリットでもありデメリットでもある要件だと思います。

## [Coder](https://coder.com/)
セルフホスト型のクラウドIDEです。Github Codespacesは基本的に無料で使用することができますが、起動時間の制限やリソースにあまり余裕がありません。しかし、Coderであればセルフホスト、つまり自宅サーバーやVPSなどにデプロイすることでホストマシンのリソースが許す限り無制限に使用することができます。

日本語資料がほとんど存在しないため、セットアップに少し手こずるかもしれませんが、構築できてしまえば完全自前かつ何も気にすることなく使用できるクラウドIDEが完成します。

無料・有料プランが存在し、有料プランだとより多くの機能にアクセスすることができます。
（ユーザーレベルのロール管理、チーム・組織単位でリソースの制限、複数Coderサーバーによる高可用性構成などなど）

![coder-pricing](/images/coder/CleanShot%202025-12-25%20at%2000.12.39@2x.png)

しかし、個人・小規模チームであれば無料プランでも十分快適に使用することができます。

### > Coderで使える機能
使える機能が多すぎるため、おすすめの機能や気になる機能に絞ってご紹介します。
詳細については公式ドキュメントを読んでみてください。

まず、大まかなメイン機能は以下の通りです。
1. **Workspaces**
    * Codespacesと同類のメイン機能です。複数コンテナを1つのワークスペースにまとめて作業することもできる？らしいです。（まだ試せていません。）
2. **Templates**
    * Workspaceを作成するためのテンプレートです。基本的には`Terraform`で記述した`main.tf`ファイルで構成され、ワークスペース作成時にビルドが走り毎回決まった形の環境を構築できます。
    * テンプレートには`modules`というCoder特有の拡張機能を組み込むことができ、`vscode-web`や`VNC`など様々な機能が公式に配布されています。モジュールで使用されているのは大元はテンプレートと同じ`.tf`ファイルなので自作・組み込みをすることも可能です。
3. **Tasks**
    * ※まだ使用できていません。β版です。
    * 公式ドキュメントによると、`Claude Code`のようなCLI系のエージェントや`AWS Bedrock`, `Google Cloud | VertexAI`のようなAPIキーを使用するタイプのLLM、また、ローカル（Ollama）などで動作するOpenAI互換のAPIを使用することでCoder上で決まったタスクを実行してくれる機能だと思います。
    * Task用Workspaceで作業を行うため、実環境は汚染することなくエージェントがタスクをこなしてくれるためCoderのメリットを最大限活かした機能になります。

### > Templateについて
公式に配布されているテンプレートだけでも非常にたくさんあります。基本的には`Docker Containers`をCoderで稼働させ使用することになると思われますが、`AWS`, `Azure`の外部クラウド環境もリモートで使用可能なのだと思います。
![coder-templates](https://raw.githubusercontent.com/coder/coder/v2.29.1/docs/images/admin/templates/starter-templates.png)

配布テンプレートをそのまま使用することも可能ですが、本当に必要最低限な機能のみであり、すごくシンプルなDocker Containerです。そのため、Coderを活かして本格的に開発環境構築をするならDockerテンプレートをベースに`Dockerfile`, `main.tf`を個人で作成するのがベストです。

また発展途上ですが、[Krz-Techのリポジトリ](https://github.com/Krz-Tech/coder)上にテンプレートを保存しているためいつか更新されるかもしれません👀

> 他にもたくさん機能はありますが、日本語ドキュメントが少なすぎて全体をテストできていないため今回はカットします。

## Coderのインストール
インストールは実は超簡単です。Linux / MacOS環境の方は以下のスクリプトを実行するだけで簡単にインストールできます。
このスクリプトでインストールするのはCoder CLI / Coder Server両方を含むため、サーバーにもクライアントにもなります。
```bash
# Linux / MacOS
curl -L https://coder.com/install.sh | sh
```

Windows環境は公式GithubのReleasesに`.msi`, `.exe`があるそうなのでそれを実行すればインストールすることができます。

wingetがある方は以下のコマンドでインストールできます。
```bash
winget install Coder.Coder
```

---

本番環境・k8s環境など本格的に運用したい方、クラウドで運用したい方は以下のようにそれぞれ方法が記載されているのでドキュメントは必見です。
https://coder.com/docs/@v2.29.1/install

![coder-install](/images/coder/CleanShot%202025-12-25%20at%2000.38.59@2x.png)

> 構築済みのCoderサーバーに既に接続している場合は右上のアカウントから`Install CLI`をクリックすることでログインまで自動でしてくれるインストールスクリプトを用意してくれます。
> ![coder-install-cli](/images/coder/CleanShot-2025-12-25-at-00.43.24@2x.png)

---

インストールが完了したら
```bash
coder server
```
でCoderサーバーが起動します。

サーバーとしてではなく、Coder CLIのクライアントとしてCoderに接続したい場合は
```bash
coder login https://coder.example.com
```
のようにCoderのデプロイ先URLを入力してログインすることができるためこちらについても非常にシンプルで使いやすいです。素晴らしいですね。

### > Coderの公開のやり方
個人的に最も楽でセキュアなのは`Cloudflare Tunnel + Coder`の構成です。
Cloudflare Tunnelを使用することでAccessを使って二重でアクセス管理をすることもできますし、トンネル経由なため自宅鯖で運用している際には自宅ネットワークに穴を開ける必要がなくなります。
後述するワイルドカードドメイン設定にもCloudflareを使うことが望ましいため、総合してCloudflareとの組み合わせが最適だと考えられます。（他に良い構成がある人教えてください👀）

Coderは環境変数でサーバーの設定をすることができます。直接`export`で触ることが公式で説明されていますが、私は既存の`/etc/coder.d/coder.env`に直接記述して`systemctl restart coder`で再読み込みさせています。Coder環境からホストマシンにアクセスすることは結構難しいと思うのでこれでも大丈夫なはず...?

---

Cloudflare Tunnelを使って外部公開をすると想定した場合、最低限必要な環境変数の設定は以下の通りです。

```env
CODER_HTTP_ADDRESS=0.0.0.0:80
CODER_TLS_ENABLE=true
CODER_TLS_ADDRESS=0.0.0.0:443

CODER_ACCESS_URL=https://coder.example.net # これがアクセスURLになります。
CODER_REDIRECT_TO_ACCESS_URL=true
```

`0.0.0.0`にしているのは単に面倒なだけなので自宅の構成によって変更してください。どちらにせよ`CODER_REDIRECT_TO_ACCESS_URL`が有効になっている時点で絶対にアクセスURLに飛ばされます。

#### >> ワイルドカードドメインを設定
こちらに関してはワイルドカードドメイン用の証明書が取得可能な人向けです。
ワイルドカードドメイン設定をすることでよりCodespacesに近い使い方をすることができます。

例えばワークスペース内で`FastAPI`や`Vite`などのWeb系開発サーバーを稼働させた場合、`https://xxxx-workspace.coder.example.net/`のように自動でHTTP / HTTPSサービスを公開してくれます。自己証明書をサーバーに設置しTLS認証を行うこともできますが、Cloudflare Tunnelを使用している場合厄介なことになるため基本設定しなくてもセキュアに動いてくれます。 

ワイルドカードを設定せずにポートフォワードを行った場合は`https://coder.example.net/xxx_user/workspace/8080`のようなパス形式のルーティングになり、場合によっては正常に動作しません。Web経由のポートフォワードを使用したい方はワイルドカード証明書を取得することをおすすめします。

しかし、VSCodeの拡張機能やCoder Desktopなどトンネル接続を行う場合は話が別で、そのままポートフォワードが行われるため正常に`localhost`で接続できるようになります。
必要に応じて機能は分別すると良いと思います。

### > ユーザー認証について
ユーザー認証は複数の方法があります。デフォルトでは`Github, パスワード認証`ですが、ベストは`Github Authentication`を使用した`Organization`, `Team`単位の認証だと思います。

後述しますが、テンプレートのモジュールの一部にGithubのログイン機能を使用して自動的にSSH鍵をアップロードする機能があったりするため、他のログインプロバイダを使用するよりGithubを使用するのが最もシンプルかつ利便性の高い構成になると考えられます。

![coder-login-github](/images/coder/CleanShot%202025-12-25%20at%2001.06.38@2x.png)

`Github Authentication`の設定は[公式ドキュメント](https://coder.com/docs/@v2.29.1/admin/users/github-auth)を読んだら結構簡単に実装できるため今回は省略します。`Organization`で認証するならそこで`Github App`を作成して環境変数にシークレットをブチ込むだけです。

### > テンプレートの作り方について
テンプレートを作成する方法は2種類あり、1つはCoder上のエディタを使用する方法、2つはCoder CLIからpushする方法です。

個人的にはテンプレート管理用リポジトリを作成し、そこからCoder CLIでpushするのが一番楽だと思います。

自分は以下みたいなディレクトリ構造で各テンプレートを管理して`setup.sh`を実行したらファイルの中身の設定に応じてテンプレート名・タグ・アイコンなど色々設定してくれるようにしてます。
```
example-template/
├--coder/
│  └ main.tf
│  └ Dockerfile
└--setup.sh
```
```sh:setup.sh
#!/bin/bash

NAME="docker-containers-example"

# Optional
DISPLAY_NAME="Docker Containers | Example"
DESCRIPTION="Docker Container Example | ZennZenn"
ICON="/icon/zenn.svg"
TAGS="[docker, containers, zenn]"

#= = = = = = = = = =
# Scripts
#= = = = = = = = = =

PREFIX="Coder | ${NAME} >>"

PushTemplate () {
    echo "${PREFIX} Pushing template"
    coder templates push -d coder -y $NAME
}

UpdateTemplateMetadata () {
    echo "*" * 10
    echo "${PREFIX} Updating template metadata"
    coder templates edit --display-name "$DISPLAY_NAME" --description "$DESCRIPTION" --icon "$ICON" "$NAME" 
}

PushTemplate
UpdateTemplateMetadata
```
※劇ガバスクリプト

---

テンプレート自体は`Terraform`で`main.tf`に記述します。筆者はTerraformに関する知識はあまりありませんが、配布テンプレートを触ってたらいい感じに動いてしまったレベルなので難しくはありません。
設定ファイルの一部に書いてありますが、使用するDockerイメージを指定することができるのですが、`Dockerfile`を設定してワークスペース作成時にビルドさせることもできます。

個人的には`Dockerfile`を作成・ビルドする方がおすすめです。公式イメージでは英語だったり結構なパッケージが欠落しており、Coderを使用して最適な開発環境を構築するには向いていない気がします。

直近ではハッカソンのバックエンド開発で使用しましたが、`ubuntu:latest`をベースに必要なパッケージをインストールしたり日本語環境を構築したりしてました。

また、Dockerfileを使用したビルド以外にも`main.tf`のファイル上あたりにある、`startup_scripts`にワークスペース起動時に実行させるシェルスクリプトを記述できます。
イメージに含めるほどではない依存関係インストールや簡単な確認をこちらで行えます。ワークスペース作成から初回起動までのフラグが環境変数で設定されているため、初回のみ実行させるみたいなこともできます。

#### >> [モジュール](https://registry.coder.com/modules)について
かなり上の部分で記述していましたが、テンプレートにはモジュールを含めることができます。
モジュールには様々な種類があり、`VSCode Web`や`Git Clone`など色々な便利な機能が含まれています。
これらは種類によりますが、基本的に`coder_app`としてワークスペースに表示され、1クリックでVSCode Webを起動したり、ローカルのIDEを起動し接続したりできます。

##### >>> おすすめモジュール
1. [VS Code Web](https://registry.coder.com/modules/coder/vscode-web)
    * ベストなやつ。
    * CodespacesみたいなWeb版VSCodeを使用することができる。
    * code-serverと違いちゃんとVSCodeなので拡張機能やログインなど色々できる。
    * code-serverも内蔵されているのでローカルのVSCodeから接続することもできる。
2. [Git Clone](https://registry.coder.com/modules/coder/git-clone), [Git Config](https://registry.coder.com/modules/coder/git-config), [Github Upload PublicKey](https://registry.coder.com/modules/coder/github-upload-public-key)
    * Githubを使ってワークスペースの初期化をするならマスト。
    * 特にGithub Upload PublicKeyは設定しておけば自動的にSSH鍵をプッシュしてくれるため、簡単にGitが使えます。
    * Git Cloneはクローン完了時に指定スクリプトを実行させることも可能。
3. 各IDEモジュール
    * 普段使っているIDEのモジュールを入れるのがおすすめ。
    * IDEに[Coder拡張機能](https://marketplace.visualstudio.com/items?itemName=coder.coder-remote)をインストールしておけばリンクをクリックすると自動的にトンネルに接続してくれる。
4. [KasmVNC](https://registry.coder.com/modules/coder/kasmvnc)
    * Coder特有のおもしろモジュール
    * テンプレートで構築できるLinuxデスクトップ環境にWebブラウザから簡単に接続できるVNCを作れる。
    * やろうと思えばどんなOSでもGUIを動かせる。（遅延はそこそこある。VNCの限界。）

その他愉快なモジュールがたくさんあるのでぜひ見てください。
>（ここには書いていませんが、Dockerで動くWindows環境を用意するとワークスペース作成でWindowsが動く謎環境作れます。（実践済み））

### > ワークスペースの作成
テンプレートが追加されていれば簡単にワークスペースは作成できます。
ワークスペース画面の`New Workspace`からテンプレートを選択し、名前を入力し`Create Workspace`するだけで完了です。Coderエージェントが作業を終えるまで待ちましょう。

![coder-workspace-create-1](/images/coder/CleanShot%202025-12-25%20at%2001.56.59@2x.png)
アイコンとタイトルうまく設定するとわかりやすい

![coder-workspace-create-2](/images/coder/CleanShot%202025-12-25%20at%2001.58.26@2x.png)
起動画面がかっこいい

![coder-workspace-create-3](/images/coder/CleanShot%202025-12-25%20at%2001.58.57@2x.png)
起動完了。`coder_app`が表示される。押したらそれぞれの機能が呼び出される。UI見やすくていい。

#### >> ワークスペース内について
コンテナとして起動しているため外部アクセスはちょっとめんどくさいです。ただCoderトンネルによって基本的にはトンネリングできるため、開発環境として使用するのであれば特に問題は発生しないと思います。

#### >> AIエージェントについて
Coder内部でエージェントを使うには個別にログインする必要があります。しかし、AntigravityやCursorのようなローカルIDEにトンネルから接続することでネイティブに使用することができるため快適です。WindowsであってもIDEから接続すればいつも通りだけどクラウド環境で作業が可能なため動作の問題を気にする必要がありません。

## まとめ
* Coderは自宅サーバーを持っている人たちにはすごく良いプロダクトである。
* Codespacesの便利さは理解してるけどリソースに余裕がないことに対して不満を感じている人向け。
* チーム・プロジェクトで統一された開発環境を快適かつ簡単に構築したい人向け。
* **インフラ好きに刺さる。**

---

# 以下現在進めている活用例紹介です。Minecraftサーバー開発に関わっている方はぜひ読んでください。
## Coderを使ったMinecraftサーバー開発
Minecraftサーバー開発の手法には様々な手段があると思います。大体の方が`ローカルマシンで開発→開発サーバーへ→本番環境へ`みたいなフローを取っていると思いますが、Coderを使った活用を現在試験運用しています。

通常のWebアプリケーション開発であれば、`HTTP / HTTPS`通信なため、Cloudflare Tunnelを使用したネイティブなトンネルが使用できますが、`TCP`には対応していません。
しかし、Coderにはトンネルを使用したIDEへのポートフォワード機能があり、それを使用すれば`TCP`であっても転送することができます。

Minecraft Java EditionはTCPでサーバーと通信しているため、この手法を使用することで`Coder -> Coder Port-Forward -> Cloudflare Tunnel -> Coder CLI (IDE) -> Minecraft`といった通信経路を通ることができ、クラウド開発環境だけどMinecraftは`localhost`で接続できるおもしろ環境が構築できます。

実際に運用しながら開発している感想ですが、かなり快適であり、**Minecraftサーバーを運営・開発してる人なんて大体自作鯖勢（偏見）だと思うのでぜひ導入してみて環境を試してみてください。** 質問等あればTwitterなどに飛ばしていただければ多分返します。


# あとがき
* AI Tasksがめちゃくちゃ面白そう。エージェントをバックグラウンドでワークスペースにアタッチして作業させられそう？いつか試す。
* 日本語化をしようか検討中。