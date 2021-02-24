---
lab:
    title: 'ラボ: Azure で Chef を使用してアプリをデプロイする'
    az400Module: 'モジュール 14: Azure で利用可能なコード ツールとしてのサード パーティ インフラストラクチャ'
---

# ラボ: Azure で Chef を使用してアプリをデプロイする
# 学生用ラボ マニュアル

## ラボの概要

このラボでは、Chef Server を使用して PartsUnlimted MRP アプリケーション (PU MRP アプリ) を Azure VM にデプロイします。このラボは、Azure VM ベースのデプロイにおける Chef の主要な機能と能力を説明します。 

Chef では、次のデプロイ オプションをサポートしています。

- Chef Server として構成されており、サポートされている Linux 配布。この目的で、標準的な Ubuntu イメージに基づき、Azure VM を使用できます。
- 事前構成されている Chef Automate イメージ。Chef Automate イメージに基づいて Azure VM をデプロイできます。イメージには Chef サーバーが含まれています。
- ホステッド Chef サーバー。ホステッド Chef サーバー アカウントは https://manage.chef.io でサインアップし、ホステッド サーバーを使用して Chef 環境を構成できます。

このラボでは、Chef Automate イメージに基づいて Azure VM をデプロイし、無料の 30 日間のトライアル ライセンスを活用します。 

ラボは以下の高レベルの手順で構成されます。

- Azure で Chef Automate サーバーをデプロイする。
- ワークステーションを構成して、Chef Automate サーバーを接続して操作する。
- インストールを自動化するため、PU MRP アプリケーションとその依存関係のクックブックとレシピを作成する。
- クックブックを Chef Automate サーバーにアップロードする。
- ロールを作成し、複数のサーバーに適用できるクックブックと属性のベースラインを定義する。
- ノードを作成し、構成を Linux VM にデプロイする。
- ノード Linux VM をブートストラップし、Chef に依存して PU MRP アプリケーションをデプロイするロールを追加する。

## ラボ環境

ラボ環境は 2 つの Azure VM とラボのコンピューターで構成されます。Azure VM には以下が含まれます:

- Chef サーバーを格納する Chef Automate
- Chef を使用して PU MRP アプリケーションをデプロイする Chef ノード。

Azure Resource Manager テンプレートを使用して、2 つの Azure VM をデプロイします。ラボのコンピューターは、Chef サーバーに接続して管理するために使用する Chef ワークステーションとして機能します。Chef ワークステーションは Windows Server またはクライアント オペレーティング システム、Linux、または MacOS の最新バージョンを実行できます。 

## 目標

このラボを完了すると、次のことができるようになります。

- Azure Resource Manager テンプレートを使用して Chef Automate サーバーとクライアントをデプロイする
- Chef ワークステーションを構成する
- Chef クックブックとレシピを作成し、Chef Automate サーバーにアップロードする
- Chef のロールを作成してクライアントにデプロイする

## ラボの所要時間

-   推定時間: **90 分**

## 指示

### 開始する前に

#### ラボの仮想マシンにログインする

必ず次の資格情報を使用して Windows 10 コンピューターにサインインしてください。
    
-   ユーザー名: **Student**
-   パスワード: **Pa55w.rd**

#### このラボで必要なアプリケーションのレビュー

このラボで使用するアプリケーションを特定:
    
-   Microsoft Edge

#### Azure サブスクリプションの準備

-   既存の Azure サブスクリプションを識別するか、新しいものを作成します。
-   Azure サブスクリプションでは所有者のロール、Azure サブスクリプションに関連のある Azure AD テナントではグローバル管理者のロールで Microsoft アカウントまたは Azure AD アカウントがあることを確認します。詳細については、[Azure portal を使用して Azure ロールの割り当てを一覧表示する](https://docs.microsoft.com/ja-jp/azure/role-based-access-control/role-assignments-list-portal) および [Azure Active Directory で管理者ロールを表示して割当てる](https://docs.microsoft.com/ja-jp/azure/active-directory/roles/manage-roles-portal#view-my-roles) を参照してください。

### 演習 1: Chef サーバーを使用して PartsUnlimted MRP アプリケーションを Azure VM にデプロイ 

この演習では、Chef サーバーを使用して PartsUnlimted MRP アプリケーションを Azure VM にデプロイします。

#### タスク 1: Chef Automate サーバーを Azure VM にデプロイする

このタスクでは、Azure Resource Manager テンプレートを使用して Azure VM をデプロイして構成します。Azure VM は Chef Automate サーバーとして機能します。

1.  ラボのコンピューターから Web ブラウザーを起動し、[**Azure Portal**](https://portal.azure.com) に移動します。このラボで使用している Azure サブスクリプションで少なくとも共同作成者のロールがあるユーザー アカウントを使ってサインインします。
1.  別のブラウザー タブを開き、[Azure Resource Manager テンプレートのリンク](
https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FPartsUnlimitedMRP%2Fmaster%2FLabfiles%2FAZ-400T05-ImplemntgAppInfra%2FLabfiles%2FM04%2FDeployusingChef%2Fenv%2Fdeploychef.json). これで自動的に Azure portal の 「**カスタム デプロイ**」 ブレードにリダイレクトされます。
1.  Azure portal の 「**カスタム デプロイ**」 ブレードで、以下の設定を指定します (他の設定は既定値のままにします):

    | 設定 | 値 |
    | --- | --- |
    | 定期売買 | このラボで使用する Azure サブスクリプションの名前 |
    | リソース グループ | 新しいリソース グループ「**az400m14l02a-RG**」の名前 |
    | リージョン | Azure VM をデプロイできる Azure リージョンの名前 |
    | 管理者ユーザー名 | **azureuser** |
    | 管理者のパスワード | **Pa55w.rd1234** |
    | 認証の種類 | **password** |
    | 診断ストレージ アカウント名 | 文字で始まり、3 ～ 24 字の文字と数字で構成された一意の文字列 |
    | 診断ストレージアカウント (新規または既存) | **new** |
    | 場所 | Azure VM をデプロイする場合と同じ Azure リージョンの名前 (スペースなし) |
    | パブリック IP アドレス名 | **az400m14l02b-pip** |
    | パブリック IP Dns 名 | 文字で始まり、3 ～ 63 字の文字と数字で構成された一意の文字列 |
    | ストレージ アカウント名 | 診断ストレージ アカウントと同じ名前 |
    | 仮想ネットワーク名 | **az400m14l02b-vnet** |
    | Vm 名 | **az400m14chs-vm** |

1.  「**カスタム デプロイ**」 ブレードで 「**レビューと作成**」 をクリックします。検証プロセスが完了していることを確認して、「**作成**」 をクリックします。

    > **注**: デプロイが完了するのを待ちます。これにはおよそ 10 分かかります。

1.  デプロイの完了後、Azure portal でページの最上部にある 「**リソース、サービス、ドキュメントの検索**」 テキスト ボックスを使い、**仮想マシン** リソースを検索して選択します。「**仮想マシン**」 ブレードで、新しくデプロイされた **az400m14chs-vm** 仮想マシンを示すエントリを選択します。
1.  **az400m14chs-vm** ブレードで、マウスのポインターを 「**DNS 名**」 エントリの上に動かし、これをクリップボードにコピーします。 
1.  新しいブラウザー タブを開き、クリップボードにコピーした DNS 名に移動します。**Https** プレフィックスを使用します。 

    > **注**: 証明書エラーは無視してください。これは予測されていることです。 

1.  Chef Automate ログイン プロンプトにアクセスできることを確認します。

    > **注**: Chef Automate にサインインする前に次のタスクを完了する必要があります。 

#### タスク 2: Chef ワークステーションを構成する

このタスクでは、Chef スターター キットをインストールし、ラボのコンピューターを Chef ワークステーションとして構成します。Chef ワークステーションから Chef サーバーに接続して管理します。

1.  ラボのコンピューターで別の Web ブラウザー ウィンドウを起動し、[Chef 開発キット (Chef DK)](https://downloads.chef.io/chefdk) ダウンロード ページに移動し、ラボの仮想マシン (Windows 10) に一致する Chef 開発キットのバージョンを選択します。「**ダウンロード**」 をクックし、「**ChefDK のダウンロード**」 ポップアップ ウィンドウで、要求された連絡先情報を提供します。再び、「**ダウンロード**」 をクリックします。

    > **注**: 2020 年 12 月時点で、Chef 開発キットの最新バージョンは 4.12.0 です。Windows 10 向けの Chef 開発キット 4.12.0-1-x64 は、このラボでテストされています。

1.  Chef 開発キットのインストーラーをダウンロードした後、既定の設定のインストールを完了し、その結果生じたデスクトップのショートカットをダブルクリックして 「**管理者: ChefDK**」 PowerShell ウィンドウを起動します。 
1.  「**管理者: ChefDK**」 PowerShell ウィンドウで以下を実行し、インストールを確認します:

    ```
    chef verify
    ```

1.  製品のライセンスを承諾するよう指示されたら、「**yes**」と入力して **Enter** キーを押します。
1.  確認できなければ、「**Administrator: ChefDK**」 PowerShell ウィンドウで以下を実行し、名前をメール アドレスのあるグローバル Git 変数を構成してから再び `chef verify` を再実行します: 

    ```
    git config --global user.name "azureuser"
    git config --global user.email "azureuser@partsunlimitedmrp.local"
    ```

    > **注**: Git を使用して Chef 構成の詳細を格納します。

    > **注**: PowerShell ウィンドウは開いたままにします。これは後ほど、このラボで再び使用します。

    > **注**: Chef Automate を接続して管理するには、インストールをホストしている Azure VM の vmId 属性を特定する必要があります。vmId は、Azure で各 VM に割り当てられている一意の値です。vmId は、Azure PowerShell や Azure CLI など、数多くの方法で取得できます。このラボでは、この目的で PuTTY を使用します。

1.  ラボのコンピューターの Web ブラウザー ウィンドウで [PuTTY ダウンロード ページ](https://putty.org/) に移動し、PuTTY インストーラーをダウンロードします。既定の設定を使ってインストールを実行します。
1.  インストール後、「**スタート**」 メニューで **PuTTY (64-bit)** フォルダーを拡張し、「**PuTTY**」 アイコンをクリックして 「**PuTTY 構成**」 ウィンドウを開きます。
1.  「**PuTTY 構成**」 ウィンドウで 「**ホスト名 (または IP アドレス)**」 テキストボックスに、前のタスクの最後に特定した DSN 名を入力し、「**開く**」 をクリックします。
1.  指示されたら、「**PuTTY セキュリティ アラート**」 ウィンドウで 「**はい**」 をクリックします。
1.  指示されたら、「**PuTTY**」 コンソール ウィンドウで、**azureuser** としてサインインします (パスワードは **Pa55w.rd1234**)。
1.  サインインした後、PuTTY コンソール ウィンドウで以下を実行して vmId を特定します:

    ```bash
    sudo chef-marketplace-ctl show-instance-id
    ```

    > **注**: vmId の値を記録します。このタスクの後半で必要になります。

1.  PuTTY コンソール ウィンドウで以下を実行してセッションを終了します:

    ```bash
    exit
    ```

    > **注**: 前のタスクでインストールし、Azure VM で実行中の Chef Automate にアクセスするには、Chef スターター きっとが必要です。Chef スターター キットにアクセスするには、前のタスクの最後で特定した DNS名に /biscotti/setup を追加します。これにより、以下の形式に似た URL ができます (`<DNS_name>` プレースホルダーは、前のタスクの最後に特定した DNS 名を示します):

    ```
    https://<DNS_name>/biscotti/setup
    ```

1.  Web ブラウザー ウィンドウで、前の手順で作成した URL に移動します。証明書関係のエラーは無視し、「**セットアップ承認**」 ページに進みます。 
1.  「**セットアップ承認**」 ページで、**vmId** を提供するよう指示されたら、**PuTTY** セッション内で特定した値を入力し、「**承認**」 をクリックします。
1.  Chef Automate セットアップの詳細を提供するよう指示されたら、以下の設定を指定して 「**Chef Automate の設定**」 をクリックします:

    | 設定 | 値 |
    | --- | --- |
    | 名を入力します | **Azure** |
    | 姓を入力します | **ユーザー** |
    | ユーザー名を入力します | **azureuser** |
    | メールを入力します | **azureuser@partsunlimitedmrp.local** |
    | パスワードを入力します | **Pa55w.rd1234** |
    | サポート アカウントを作成し、Chef Automate サブスクリプションに含まれている Chef ベース サポートを有効にします | **オフ** |
    | EULA およびマスター合意書を読み、承諾します | **オン** |

    > **注**: これは**既定**の組織名に適用されます。Chef Automate 構成セットアップの一部として組織名を指定しないためです。オプションで、Chef Automate サーバー内で組織名を指定することもできます。この組織の値は、このラボで後ほど、**knife.rb** ファイルに含まれます。 

1.  指示されたら、「**スターター キットのダウンロード**」 をクリックします。これにより、starter-kit.zip という名前のファイルのダウンロードがトリガーされます。この中には、事前に構成され、Chef のセットアップを促すファイルが含まれています。 

    > **注**: Chef スターター キットには、ユーザー資格情報と証明書の詳細が含まれています。最初の登録中に異なる資格情報と詳細が生成されます。これは提供されるユーザーと組織の詳細に固有のものになります。Chef スターター キットは複数のユーザーまたは登録で再使用しないでください。

    > **注**: Chef スターター キットのダウンロード後、「**Chef Automate にログイン**」 ボタンを利用できるようになります。 

1.  Web ブラウザーウィンドウで 「**Chef Automate にログイン**」 ボタンをクリックし、Chef Automate にサインインします。「**Chef Automate セットアップ**」 Web ページで提供したユーザー名 (**azureuser**) とパスワード (**Pa55w.rd1234**) を使用します。これで、Chef Automate ダッシュボードが表示されます。
1.  ラボのコンピューターでエクスプローラーを開き、**Chef スタート キット** zip アーカイブの内容を **Downloads** フォルダーから **C:\\Labfiles\\chef** フォルダーに抽出します。
1.  エクスプローラーで **C:\\Labfiles\\chef\\chef-repo\\.chef** フォルダーに移動し、メモ帳で **knife.rb** ファイルを開きます。 
1.  **Knife.rb** ファイルで、**chef_server_url** 名に含まれている完全修飾ドメイン名が前のタスクで特定された Azure VM DNS 名に一致しており、組織名が**既定**に設定されていることを確認し、メモ帳ウィンドウを閉じます。
1.  「**管理者: Chef DK**」 PowerShell ウィンドウに戻り、以下を実行して新しいリポジトリを開始します:

    ```powershell
    Set-Location -Path 'C:\Labfiles\chef\chef-repo'
    git init
    git add -A
    git commit -m "starter kit commit"
    ```

    > **注**: この Chef サーバーには、信頼されていない SSL 証明書が含まれています。このため、Chef ワークステーションの SSL 証明書を手動で信頼し、Chef サーバーと通信できるようにする必要があります。これを変更するため、以下のコマンドを実行して Chef の有効な SSL 証明書をインポートします:

    ```chef
    knife ssl fetch
    ```

1.  「**管理者: Chef DK**」 PowerShell ウィンドウで以下を実行し、chef-repo の現在の内容を一覧表示します:

    ```powershell
    Get-ChildItem -Path '.\' -Recurse
    ```

1.  「**管理者: Chef DK**」 PowerShell ウィンドウで以下を実行し、Chef ワークステーション リポジトリの内容を Chef Automate Azure VM ベースのリポジトリの内容に同期させます: 

    ```chef
    knife download /
    ```

    > **注**: このコマンドは、Chef Automate サーバーから Chef リポジトリ全体をダウンロードします。

    > **注**: ユーザー アカウントの詳細によっては acls に関連したエラーが表示されるかもしれませんが、このラボの目的上、これらのエラーは無視できます。

1.  「**管理者: Chef DK**」 PowerShell ウィンドウで再び以下を実行し、`knife download /` コマンドを実行した後、chef-repo の最新の内容を一覧表示し、chef-repo ディレクトリで作成された追加ファイルとフォルダーをメモします。

    ```powershell
    Get-ChildItem -Path '.\' -Recurse
    ```

1.  「**管理者: Chef DK**」 PowerShell ウィンドウで以下を実行し、新しく追加されたファイルを Git リポジトリ にコミットします:

    ```git
    git add -A
    git commit -m "knife download commit"
    ```

#### タスク 3: Chef クックブックとレシピを作成し、Chef Automate サーバーにアップロードする

このタスクでは、PU MRP アプリの依存関係向けにクックブックとレシピを作成して、PU MRP アプリケーションのインストールを自動化し、このクックブックとレシピを Chef サーバーにアップロードします。

1.  「**管理者: Chef DK**」 PowerShell ウィンドウで以下を実行し、Chef ナイフ ツールを使用して **chef-repo** の **cookbooks** サブディレクトリでクックブック テンプレートを生成します:

    ```chef
    Set-Location -Path '.\cookbooks'
    chef generate cookbook mrpapp
    ```

    > **注**: クックブックは、アプリケーションまたは機能を構成するタスクのセットです。。シナリオと、そのシナリオをサポートするために必要なものをすべて定義します。クックブック内には、クックブックが実行するアクションを定義する一連のレシピがあります。クックブックとレシピは Ruby 言語で書かれています。実行した chef 生成クックブック コマンドにより、**mrpapp** ディレクトリが **chef-repo\\cookbooks** ディレクトリで作成されます。The mrpapp ディレクトリには、クックブックと既定のレシピを定義する定型コードがすべて含まれています。

1.  「**管理者: Chef DK**」 PowerShell ウィンドウで以下を実行し、編集できるように **metadata.rb** ファイルを開きます。

    ```powershell
    notepad .\mrpapp\metadata.rb
    ```

    > **注**: クックブックとレシピは他のクックブックやレシピを利用できます。このクックブックは既存のレシピを使用して、APT リポジトリを管理します。 

1.  **metadata.rb** ファイルの内容が表示されているメモ帳ウィンドウで、以下のラインをファイルの最後に追加し、保存してからメモ帳ウィンドウを閉じます。

    ```chef
    depends 'apt'
    ```

    > **注**: 次に、レシピの 3 つの依存関係をインストールする必要があります。**apt** クックブック、**windows** クックブック、**chef-client** クックブックです。3 つのクックブックは [公式 Chef クックブック リポジトリ](https://supermarket.chef.io/cookbooks)からダウンロードし、`knife cookbook site` コマンドを使用してインストールします。

1.  「**管理者: Chef DK**」 PowerShell ウィンドウで以下を実行し、`knife cookbook site` コマンドを使用してクックブックをダウンロードしてインストールします:

    ```chef
    knife cookbook site install apt
    knife cookbook site install windows
    knife cookbook site install chef-client
    ```

    > **注**: 次に、**default.rb** レシピの完全な内容を [Microsoft Parts Unlimited MRP GitHub リポジトリ](https://raw.githubusercontent.com/Microsoft/PartsUnlimitedMRP/master/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/M04/DeployusingChef/final/default.rb)からコピーする必要があります。

1.  ラボのコンピューターで別の Web ブラウザー ウィンドウを起動し、**https://raw.githubusercontent.com/Microsoft/PartsUnlimitedMRP/master/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/M04/DeployusingChef/final/default.rb** に移動して RAW 形式で **default.rb** レシピを表示します。Web ページの内容をクリップボードにコピーします。
1.  「**管理者: Chef DK**」 PowerShell ウィンドウで以下を実行し、**C:\\Labfiles\\chef\\chef-repo\\cookbooks\\mrpapp\\recipes\\default.rb** ファイルをメモ帳で開きます:

    ```powershell
    notepad .\mrpapp\recipes\default.rb
    ```

1.  メモ帳ウィンドウでファイルの内容をレビューし、以下のように見えることを確認します:

    ```chef
    #
    # Cookbook:: mrpapp
    # Recipe:: default
    #
    # Copyright:: 2020, The Authors, All Rights Reserved.
    ```

1.  メモ帳ウィンドウで、クリップボードの内容をファイルの最後に追加します (新しいラインを始めます)。変更を保存してメモ帳ウィンドウを閉じます。

    > **注**: 以下は、コピーしたレシピがアプリケーションをプロビジョニングする方法を説明し、**default.rb** のラインごとの内容の意味を説明します。

    > **注**: レシピは `apt` リソースを実行します。これにより、レシピは実行前に `apt-get update` コマンドを実行します。このコマンドでは、ローカル パッケージが最新バージョンであることを確認します:

    ```chef
     	# Runs apt-get update
     	include_recipe "apt"
    ```

    > **注**: 次に、`apt_repository` リソースを追加して、**OpenJDK** リポジトリが apt リポジトリの一部であり、最新の状態であることを確認します:

    ```chef
 	# Add the Open JDK apt repo
 	apt_repository 'openJDK' do
 		uri 'ppa:openjdk-r/ppa'
 		distribution 'trusty'
 	end
    ```

    > **注**: 次に、`apt-package` レシピを使用して、**OpenJDK** と **OpenJRE** がインストールされていることを確認します。使用したくないレガシー パッケージに応じて、ヘッドレス バージョンをフル バージョンとして使います。

    ```chef
 	# Install JDK and JRE
 	apt_package 'openjdk-8-jdk-headless' do
 		action :install
 	end

 	apt_package 'openjdk-8-jre-headless' do
 		action :install
     	end
    ```

    > **注**: 次に、`JAVA_HOME` と `PATH` 環境変数を設定して OpenJDK を参照します:

    ```chef
 	# Set Java environment variables
 	ENV['JAVA_HOME'] = "/usr/lib/jvm/java-8-openjdk-amd64"
     	ENV['PATH'] = "#{ENV['PATH']}:/usr/lib/jvm/java-8-openjdk-amd64/bin"
    ```

    > **注**: 次に、MongoDB データベース エンジンと Tomcat Web サーバーをインストールします:

    ```chef
 	# Install MongoDB
 	apt_package 'mongodb' do
 		action :install
 	end

 	# Install Tomcat 7
 	apt_package 'tomcat7' do
 		action :install
     	end
    ```

    > **注**: この時点で、依存関係はすべてインストールされています。これでアプリケーションの構成を始められます。まず、MongoDB データベースに何らかのベースライン データがあることを確認する必要があります。`remote_file` リソースは、指定された場所にファイルをダウンロードします。このタスクは冪等なので、サーバー上のファイルのチェックサムがローカル ファイルと同じであれば、アクションは実行されません。`notifies` コマンドも使用します。これにより、リソースの実行時にファイルの新しいバージョンがある場合、指定されたリソースに通知が送られ、新しいファイルを実行するよう促します。

    ```chef
 	# Load MongoDB data
 	remote_file 'mongodb_data' do
 		source 'https://github.com/Microsoft/PartsUnlimitedMRP/tree/master/deploy/MongoRecords.js'
 		path './MongoRecords.js'
 		action :create
 		notifies :run, "script[mongodb_import]", :immediately
     	end
    ```

    > **注**: 次に、`script` リソースを使用して、前の手順でダウンロードした MongoDB データを読み込むコマンド ライン スクリプトを設定します。`script` リソースの `action` パラメーターは `nothing` に設定されています。つまり、スクリプトは、実行通知を受け取った場合にのみ実行されます。このリソースが実行されるのは、前の手順で指定された `remote_file` リソースによって通知された場合のみです。**MongoRecord.js** ファイルの新しいバージョンがアップロードされるたびに、レシピはこれをダウンロードおよびインポートします。**MongoRecords.js** ファイルが変わっていない場合は、何もダウンロードされたりインポートされたりしません!

    ```chef
 	script 'mongodb_import' do
 		interpreter "bash"
 		action :nothing
 		code "mongo ordering MongoRecords.js"
     	end
    ```

    > **注**: 次に、Tomcat が PU MRP アプリケーションを実行するポートを設定します。これは `script` リソースを使用して正規表現を呼び出し、**etc/tomcat7/server.xml** ファイルを更新します。`not_if` アクションはガード ステートメントです。`not_if` アクションのコードが `true` を返す場合、リソースは実行しません。これにより、スクリプトは確実に必要な場合にのみ実行します。

    > **注**: `#{node['tomcat']['mrp_port']}` と呼ばれる属性を参照しています。この値はまだ定義されていませんが、次のタスクで定義します。属性を使用すれば、変数を設定できるため、単一のサーバーのひとつのポート、および別のサーバーの異なるポートで PU MRP アプリケーションをデプロイできます。ポートが変わる場合は、`notifies` を使用してサービス再起動を呼び出します。

    ```chef
 	# Set tomcat port
 	script 'tomcat_port' do
 		interpreter "bash"
 		code "sed -i 's/Connector port=\".*\" protocol=\"HTTP\\/1.1\"$/Connector port=\"#{node['tomcat']['mrp_port']}\" protocol=\"HTTP\\/1.1\"/g' /etc/tomcat7/server.xml"
 		not_if "grep 'Connector port=\"#{node['tomcat']['mrp_port']}\" protocol=\"HTTP/1.1\"$' /etc/tomcat7/server.xml"
 		notifies :restart, "service[tomcat7]", :immediately
     	end
    ```

    > **注**: これで PU MRP アプリケーションをダウンロードして、Tomcat での実行を開始できます。新しいバージョンを取得した場合は、Tomcat サービスに再起動するよう合図します。

    ```chef
 	# Install the PU MRP app, restart the Tomcat service if necessary
 	remote_file 'mrp_app' do
 		source 'https://github.com/Microsoft/PartsUnlimitedMRP/tree/master/builds/mrp.war'
 		action :create
 		notifies :restart, "service[tomcat7]", :immediately
     	end
    ```

    > **注**: Tomcat サービスで希望する状態を定義できます。この特定のケースでは、サービスは実行中でなくてはなりません。これによりスクリプトは Tomcat サービスの状態をチェックし、必要であれば起動します。

    ```chef
 	# Ensure Tomcat is running
 	service 'tomcat7' do
 		action :start
 	end
    ```

    > **注**: 最後に、`ordering_service` が実行中であることを確認します。`remote_file` と `script` リソースの組み合わせを使い、ordering_service を強制終了して再起動する必要があるかチェックします。 

    ```chef
 	remote_file 'ordering_service' do
 		source 'https://github.com/Microsoft/PartsUnlimitedMRP/tree/master/builds/ordering-service-0.1.0.jar'
 		path './ordering-service-0.1.0.jar'
 		action :create
 		notifies :run, "script[stop_ordering_service]", :immediately
 	end

 	# Kill the ordering service
 	script 'stop_ordering_service' do
 		interpreter "bash"
 	# Only run when notifed
 		action :nothing
 		code "pkill -f ordering-service"
 		only_if "pgrep -f ordering-service"
 	end

 	# Start the ordering service.
 	script 'start_ordering_service' do
 		interpreter "bash"
 		code "/usr/lib/jvm/java-8-openjdk-amd64/bin/java -jar ordering-service-0.1.0.jar &"
 		not_if "pgrep -f ordering-service"
 	end
    ```

1.  「**管理者: Chef DK**」 PowerShell ウィンドウで以下を実行し、追加されたファイルを Git リポジトリにコミットします:

    ```git
    git add .
    git commit -m "mrp cookbook commit"
    ```

    > **注**: これでレシピが作成され、依存関係がインストールされたので、クックブックとレシピを Chef Automate サーバーにアップロードできます。

1.  「**管理者: Chef DK**」 PowerShell ウィンドウで以下を実行し、`knife cookbook upload` コマンドを使用してクックブックとレシピを Chef Automate サーバーにアップロードします:

    ```chef
    knife cookbook upload mrpapp --include-dependencies
    knife cookbook upload chef-client --include-dependencies
    ```

#### タスク 4: Chef ロールを作成する

この演習では、`knife` コマンドを使用してロールを作成します。ロールは、複数のサーバーに適用できるクックブックと属性のベースラインを定義します。

> **注**: 詳細については、「Chef ドキュメント ページ Knife のロール」(https://docs.chef.io/knife_role.html)を参照してください。

1.  「**管理者: Chef DK**」 PowerShell ウィンドウで以下を実行し、メモ帳で **knife.rb** ファイルを開きます:

    ```powershell
    notepad C:\Labfiles\chef\chef-repo\.chef\knife.rb
    ```

1.  メモ帳ウィンドウで、新しいラインを始めて以下をファイルの最後に追加します。変更を保存し、メモ帳ウィンドウを閉じます。

    ```chef
    knife[:editor] = "notepad"
    ```

    > **注**: このラインを追加すると、ロールを作成する際に knife.rb を開くエディターとしてメモ帳を指定します (これは次の手順で行われます)。knife.rb でエディターを指定しなければ、エディター環境変数を設定するよう指示するエラー メッセージを受け取ります。

1.  「**管理者: Chef DK**」 PowerShell ウィンドウで以下を実行してロールを作成し、**partsrole** という名前を付けます。

    ```chef
    knife role create partsrole
    ```

    > **注**: このコマンドにより、ロール定義の構造を示す JSON コンテンツが表示されたメモ帳ウィンドウが開きます。

    ```json
    {
      "name": "partsrole",
      "description": "",
      "json_class": "Chef::Role",
      "default_attributes": {

      },
      "override_attributes": {

      },
      "chef_type": "role",
      "run_list": [

      ],
      "env_run_lists": {

      }
    }
    ```

1.  メモ帳ウィンドウで、`default_attributes` セクションを以下に変更します:

    ```json
      "default_attributes": {
          "tomcat": {
              "mrp_port": 9080
          }
      },
    ```

1. override_attributes を次のように更新します:

    ```json
      "override_attributes": {
          "chef_client": {
              "interval": "60",
              "splay": "1"
          }
      },
    ```

1. run_list を次のように更新します:

    ```json
      "run_list": [
          "recipe[mrpapp]",
          "recipe[chef-client::service]"
      ],
    ```

1.  完了したファイルに以下の内容が含まれていることを確認し、変更を保存してメモ帳ウィンドウを閉じます:

    ```json
    {
      "name": "partsrole",
      "description": "",
      "json_class": "Chef::Role",
      "default_attributes": {
          "tomcat": {
              "mrp_port": 9080
          }
      },
      "override_attributes": {
          "chef_client": {
              "interval": "60",
              "splay": "1"
          }
      },
      "chef_type": "role",
      "run_list": [
          "recipe[mrpapp]",
          "recipe[chef-client::service]"
      ],
      "env_run_lists": {

      }
    }
    ```

1.  「**管理者: Chef DK**」 PowerShell ウィンドウに戻り、`knife role create partsrole` が完了していることを確認します。 

    > **注**: 必ずメモ帳ウィンドウを閉じてください。完了の成功は、`Created role「partsrole」` メッセージによって示されます。 

#### タスク 5: PU MRP アプリをブートストラップして PU MRP アプリケーションをデプロイする

この演習では、**knife** を使用して PU MRP アプリケーション サーバーをブートストラップし、これに PU MRP アプリケーション ロールを割り当てます。

> **注**: まず、Linux VM のプロビジョニングから始めます。これはその後、Chef クライアントとして構成され、前のタスクで作成したロールを使用して PU MRP アプリケーションをデプロイします。

1.  ラボのコンピューターから Web ブラウザーを起動し、[Azure Resource Manager template link](
https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FPartsUnlimitedMRP%2Fmaster%2FLabfiles%2FAZ-400T05-ImplemntgAppInfra%2FLabfiles%2FM04%2FDeployusingChef%2Fenv%2Fdeploylinux.json) に移動します。これで自動的に Azure portal の 「**カスタム デプロイ**」 ブレードにリダイレクトされます。
1.  Azure portal の 「**カスタム デプロイ**」 ブレードで、「**テンプレートの編集**」 をクリックします
1.  「**テンプレートの編集**」 ブレードのテンプレート概要ペインで 「**変数**」 セクションを拡張し、「**vmSize**」 をクリックします。
1.  テンプレートの詳細セクションで、`"vmSize": "Standard_A1",` を `"vmSize": "Standard_D2s_v3",` に変更して 「**保存**」 をクリックします。
1.  再び 「**カスタム デプロイ**」 ブレードで以下の設定を指定します (他は既定値のままにします):

    | 設定 | 値 |
    | --- | --- |
    | 定期売買 | このラボで使用する Azure サブスクリプションの名前 |
    | リソース グループ | 新しいリソース グループ「**az400m14l02b-RG**」の名前 |
    | リージョン | このラボで以前に Chef Automate Azure VM をデプロイした Azure リージョンの名前 |
    | 管理者ユーザー名 | **azureuser** |
    | 管理者のパスワード | **Pa55w.rd1234** |
    | Dns ラベル プレフィックス | 文字で始まり、3 ～ 63 字の文字と数字で構成された一意の文字列 |
    | Ubuntu OS バージョン | **16.04.0-LTS** |

1.  「**カスタム デプロイ**」 ブレードで 「**レビューと作成**」 をクリックします。検証プロセスが完了していることを確認して、「**作成**」 をクリックします。

    > **注**: デプロイが完了するのを待ちます。これにはおよそ 2 分かかります。

1.  デプロイの完了後、Azure portal でページの最上部にある 「**リソース、サービス、ドキュメントの検索**」 テキスト ボックスを使い、**仮想マシン** リソースを検索して選択します。「**仮想マシン**」 ブレードで、新しくデプロイされた **mrpUbuntuVM** という名前の仮想マシンを示すエントリを選択します。
1.  **mrpUbuntuVM** ブレードで、マウスのポインターを 「**DNS 名**」 エントリの上に動かし、これをクリップボードにコピーします。 
1.  「**管理者: Chef DK**」 PowerShell ウィンドウに戻り、以下を実行して **knife** ツールを使い、新しくデプロイされた Azure VM をブートストラップします (`<DNS_name>` プレースホルダーは、前のタスクの最後に特定した DNS 名を示します):

    ```chef
    knife bootstrap <DNS_name> --ssh-user azureuser --ssh-password Pa55w.rd1234 --node-name mrp-app --run-list role「partsrole」 --sudo --verbose
    ```
1.  接続を続行するか聞かれたら「**Y**」と入力し、**Enter** キーを押します。

    > **注**: オンボーディング スクリプトが完了するのを待ちます。これにはおよそ 3 分かかります。スクリプトは次の手順を実行します。

    - Chef クライアント コンポーネントのインストール。
    - `mrp` Chef ロールの割り当て。
    - `mrpapp` レシピの実行。

1.  スクリプトの完了後、Azure portal が表示されているブラウザー ウィンドウに切り替え、別のブラウザー タブを開きます。**http://** プレフィックスの後に `mrpUbuntuVM` Azure VM の DNS 名が続き、`:9080/mrp/` サフィックスが含まれている URL に移動します。形式は `http://<DNS_name>:9080/mrp/` のようになります (`<DNS_name>` プレースホルダーは、前のタスクの最後に特定した DNS 名を示します)。
1.  ブラウザー タブに PU MRP アプリケーションのランディング ページが表示されていることを確認します。
1.  Chef Automate サーバーのダッシュボードが表示されているブラウザー ウィンドウに切り替え、ダッシュボード ページの最上部で 「**ノード**」 をクリックします。**mrp-app** というラベルの単一のノードが含まれていることを確認します。

### 演習 2: Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。 

> **注**: 新しく作成した Azure リソースのうち、使用しないリソースは必ず削除してください。使用しないリソースを削除しないと、予期しないコストが発生する場合があります。

#### タスク 1: Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。 

1.  Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```sh
    az group list --query "[?starts_with(name,'az400m14l02')].name" --output tsv
    ```

1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを削除します。

    ```sh
    az group list --query "[?starts_with(name,'az400m14l02')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    > **注**: コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

## レビュー

このラボでは、Chef サーバーを使用して PartsUnlimted MRP アプリケーション (PU MRP アプリ) を Azure VM にデプロイする方法を学習しました。 