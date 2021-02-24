---
lab:
    title: 'ラボ: 既存の ASP.NET アプリを Azure でモダン化する'
    module: 'モジュール 15: Docker を使用したコンテナーの管理'
---

# ラボ: 既存の ASP.NET アプリを Azure でモダン化する
# 学生用ラボ マニュアル

## ラボの概要

Azure への移行の一環として Web アプリケーションをモダン化することにした場合は、アプリケーション全体を再設計する必要はありません。マイクロサービスのようにより高度なアプローチを使用したアプリケーションの再設計は、コストと時間の制約があるため、必ずしもよい選択肢ではありません。ただし、コンテナー化と Azure PaaS サービスを利用してアプリをモダン化できるかもしれません。たとえば、Azure の SQL Database のような管理対象サービスへのアプリのデータ層の移行は、接続文字列の更新に制限される可能性があり、基本的なコードを変更する必要はありません。

このラボでは、Nerd Dinner Application を使用して、このアプローチを説明します。Nerd Dinner はオープン ソース ASP.NET MVC プロジェクトです。ライブで実行中のサイトは、[http://www.nerddinner.com](http://www.nerddinner.com) からご覧いただけます。アプリケーション DB を Azure SQL インスタンスに移動し、Docker サポートをアプリケーションに追加して、Azure Container Instances でアプリケーションを実行します。

## 目標

このラボを完了すると、次のことができるようになります。

- Azure で LocalDB を SQL Server に移行する
- Visual Studio で Docker ツールを使い、アプリケーションの Docker サポートを追加する
- Docker イメージを Azure Container Registry (ACR) に公開する
- Docker イメージを ACR から Azure Container Instances (ACI) にプッシュする

## ラボの所要時間

-   推定時間: **60 分**

## 指示

### 開始する前に

#### ラボの仮想マシンにログインする

必ず次の資格情報を使用して Windows 10 コンピューターにサインインしてください。
    
-   ユーザー名: **Student**
-   パスワード: **Pa55w.rd**

#### このラボで必要なアプリケーションのレビュー

このラボで使用するアプリケーションを特定:
    
-   Microsoft Edge
-   Visual Studio 2019 Community Edition ([Visual Studio Downloads page](https://visualstudio.microsoft.com/downloads/) から入手可能)Visual Studio 2019 のインストールには、**ASP.NET および Web 開発**、**Azure 開発**、**NET Core クロスプラットフォーム開発** ワークロードを含める必要があります。これはすでにラボのコンピューターに事前インストールされています。
-   **Docker for windows** は [Docker ドキュメント サイト](https://docs.docker.com/docker-for-windows/install/#download-docker-for-windows)から入手できます。このラボでは前提条件の一部としてインストールされます。 

#### Azure サブスクリプションの準備

-   既存の Azure サブスクリプションを識別するか、新しいものを作成します。
-   Azure サブスクリプションでは所有者のロール、Azure サブスクリプションに関連のある Azure AD テナントではグローバル管理者のロールで Microsoft アカウントまたは Azure AD アカウントがあることを確認します。詳細については、[Azure portal を使用して Azure ロールの割り当てを一覧表示する](https://docs.microsoft.com/ja-jp/azure/role-based-access-control/role-assignments-list-portal) および [Azure Active Directory で管理者ロールを表示して割当てる](https://docs.microsoft.com/ja-jp/azure/active-directory/roles/manage-roles-portal#view-my-roles) を参照してください。

### 演習 0: ラボの前提条件の構成

この演習では、Nerd Dinner アプリケーションの複製や Docker Desktop の構成など、ラボの前提条件を設定します。

#### タスク 1: Nerd Dinner アプリケーションを複製する

このタスクでは、Nerd Dinner アプリケーションの複製を作成して Visual Studio で開きます。

1.  ラボのコンピューターで Web ブラウザーを起動し、[https://github.com/spboyer/nerddinner-mvc4](https://github.com/spboyer/nerddinner-mvc4) に移動します。 
1.  [Spboyer/nerddinner-mvc4](https://github.com/spboyer/nerddinner-mvc4) ページで 「**コード**」 をクリックします。ドロップダウン リストで、コード リポジトリの HTTPS URL の隣にあるクリップボードのアイコンをクリックします。
1.  ラボのコンピューターで Visual Studio を起動し、起動ページで 「**リポジトリの複製**」 をクリックします。
1.  「**リポジトリの複製**」 ページで 「**リポジトリの場所**」 テキストボックスにクリップボードの内容を貼り付け、「**複製**」 をクリックします。
1.  追加コンポーネントをインストールするよう指示されたら、「**インストール**」 をクリックします。
1.  Visual Studio ウィンドウのトップ メニューで 「**ビルド**」 をクリックします。ドロップダウン メニューで 「**ソリューションのリビルド**」 をクリックします。
1.  Visual Studio ウィンドウのトップ メニューで 「**IIS Express**」 をクリックします。Web アプリケーションが表示されている Web ブラウザーが自動的に開きます。
1.  アプリケーションがローカルで実行していることを確認し、実行中のアプリケーションが表示されている Web ブラウザーのウィンドウを閉じます。

#### タスク 2: Docker Desktop を構成する 

1.  ラボのコンピューターで Web ブラウザーを起動し、[Docker ドキュメント サイト](https://docs.docker.com/docker-for-windows/install/#download-docker-for-windows)に移動します。既定の設定で Docker Desktop for Windows をダウンロードしてインストールします。
1.  ラボのコンピューターでタスクバーを拡張し、「**Docker**」 アイコンを右クリックします。右クリック メニューで 「**Windows コンテナーに切り替える**」 オプションを選択します。
1.  確定するよう指示されたら、「**切り替え**」 をクリックします。

### 演習 1: Nerd Dinner アプリケーションをモダン化する

この演習では、Docker ベースのコンテナー化と Azure SQL Database を使用して Nerd Dinner アプリケーションをモダン化します。

#### タスク 1: Azure SQL データベースを作成する

このタスクでは、Azure SQL データベースを作成します。

1.  ラボのコンピューターから Web ブラウザーを起動し、[**Azure Portal**](https://portal.azure.com) に移動します。このラボで使用している Azure サブスクリプションで少なくとも共同作成者のロールがあるユーザー アカウントを使ってサインインします。
1.  Azure portal で、**SQL データベース** リソース タイプを検索して選択し、「**SQL データベース**」 ブレードで、「**追加**」 をクリックします。
1.  「**SQL Database の作成**」 ブレードの 「**基本**」 タブで、次の設定を指定します:

    | 設定 | 値 | 
    | --- | --- |
    | 定期売買 | お使いの Azure サブスクリプションの名前 |
    | リソース グループ | 新しいリソース グループ「**az400m15l01a-RG**」の名前 |
    | データベース名 | **nerddinnerlab** | 

1.  「**SQL データベースの作成**」 ブレードの 「**基本**」 タブで、「**サーバーの選択**」 ドロップダウン リストのすぐ下にある 「**新規作成**」 をクリックします。 
1.  「**新しいサーバー**」 ブレードで以下の設定を指定し、「**OK**」 をクリックします。

    | 設定 | 値 | 
    | --- | --- |
    | サーバー名 | 有効でグローバルに一意のサーバー名 | 
    | サーバー管理者のログイン | **sqluser** |
    | パスワード | **Pa55w.rd1234** |
    | 場所 | Azure SQL データベースをプロビジョニングできるラボの場所に近い Azure リージョン |

    > **注**: サーバーで使用する名前を記録します。これは、このラボで後ほど必要になります。

1.  「**SQL Database の作成**」 ブレードの 「**基本**」 タブに戻り、「**コンピューティング + ストレージ**」 ラベルの隣で 「**データベースの構成**」 をクリックします。
1.  「**構成**」 ブレードで、「**Basic、Standard、Premium をお探しですか?**」 をクリックし、「**適用**」 をクリックします。
1.  「**SQL Database の作成**」 ブレードの 「**基本**」 タブに戻り、「**次へ: ネットワーク >**」 をクリックします:
1.  「**SQL Database の作成**」 ブレードの 「**ネットワーク**」 タブで、次の設定を指定し (他の設定は既定値のままにします)、「**追加設定**」 をクリックします。

    | 設定 | 値 | 
    | --- | --- |
    | 接続方法 | **パブリック エンドポイント** |
    | Azure のサービスとリソースにこのサーバーへのアクセスを許可する | **はい** |
    | 現在のクライアント IP アドレスを追加する | **はい** |

1.  「**SQL Database の作成**」 ブレードの 「**追加設定**」 タブで、次の設定を指定し (その他の設定は既定値のままにします)、「**レビューと作成**」 をクリックします。

    | 設定 | 値 | 
    | --- | --- |
    | 既存のデータを使用する | **None** |
    | Azure Defender または SQL | **後で** |

1.  「**SQL Database の作成**」 ブレードの 「**レビューと作成**」 タブで、「**作成**」 をクリックします。 

    > **注**: デプロイが完了するのを待ちます。これにはおよそ 5 分かかります。

#### タスク 2: Azure の SQL Server に LocalDB を移行する

このタスクでは、前のタスクで作成した Azure SQL データベースにアプリケーションの LocalDB を移行します。

1.  Visual Studio のトップ メニューで 「**表示**」 をクリックし、ドロップダウン メニューで 「**SQL Server オブジェクト エクスプローラー**」 をクリックします。
1.  「**SQL Server オブジェクト エクスプローラー**」 ペインで 「**サーバーの追加**」 アイコンをクリックします。「**接続**」 ダイアログ ボックスで以下の設定を指定し、「**接続**」 をクリックします (**<server-name>** は、前のタスクで作成した論理サーバーの名前に置き換えます):

    | 設定 | 値 | 
    | --- | --- |
    | サーバー名 | **<server-name>.database.windows.net** |
    | 認証 | **SQL Server 認証** |
    | 「ユーザー名」 | **sqluser** |
    | パスワード | **Pa55w.rd1234** |
    | データベース名 | **nerddinnerlab** | 

    > **注**: まず、LocalDB インスタンスから新しくプロビジョニングされた Azure SQL データベースにスキーマをコピーします。

1.  「**SQL Server オブジェクト エクスプローラー**」 ペインで localDB インスタンスを示す **localdb** ノードを拡張し、その **Databases** フォルダーを拡張します。  
1.  データベースのリストで、Nerd Dinner Application リポジトリに基づいて作成されたプロジェクトの一部であるデータベースを右クリックし、右クリック メニューで 「**スキーマ比較**」 をクリックします。これにより、Visual Studio ウィンドウの中央のペインで 「**SqlSchemaCompare1**」 タブが開きます。
1.  「**SqlSchemaCompare1**」 タブの最上部にある 「**ターゲットの選択**」 ドロップダウン リストで 「**ターゲットの選択**」 をクリックします。 
1.  「**ターゲット スキーマの選択**」 ダイアログ ボックスで 「**データベース**」 オプションが選択されていることを確認して 「**接続の選択**」 をクリックします。「**接続**」 ダイアログ ボックスで **nerddinnerlab** Azure SQL データベースを選択し、「**接続**」 を選択します。「**ターゲット スキーマの選択**」 ダイアログ ボックスに戻り、「**OK**」 をクリックします。
1.  Visual Studio ウィンドウの中央のペインにある 「**SqlSchemaCompare1**」 タブで 「**比較**」 をクリックします。比較が完了するまで待ちます。
1.  比較が完了したら、「**SqlSchemaCompare1**」 タブで 「**更新**」 をクリックし、確定するよう指示されたら 「**はい**」 をクリックします。
1.  「**SqlSchemaCompare1**」 タブを閉じます。

    > **注**: LocalDB インスタンスから、新しくプロビジョニングされた Azure SQL データベースにデータをコピーします。

1.  「**SQL Server オブジェクト エクスプローラー**」 ペインで 「LocalDB インスタンス」 を右クリックし、右クリック メニューで 「**データ比較**」 をクリックします。「**新しいデータの比較**」 ウィザードが起動し、「**SqlDataCompare1**」 タブが開きます。
1.  「**新しいデータの比較**」 ウィザードの 「**ソース データベースとターゲット データベースの選択**」 ページにある 「**ターゲット データベース**」 セクションで、「**接続の選択**」 をクリックします。「**接続**」 ダイアロブ ボックスで **nerddinnerlab** Azure SQL データベースを選択し、「**接続**」 をクリックします。「**ソース データベースとターゲット データベースの選択**」 ページに戻り、「**次へ**」 をクリックします。
1.  「**比較するテーブル、フィールド、およびビューの選択**」 で、「**テーブル**」 と 「**ビュー**」 エントリの隣にあるチェックボックスを選択し、「**終了**」 をクリックします。
1.  「**SqlDataCompare1**」 タブのトップ メニューで 「**ターゲットの更新**」 をクリックし、確定するよう指示されたら 「**はい**」 をクリックします。
1.  「**SqlDataCompare1**」 タブを閉じます。

    > **注**: コード変更を避けるため、**web.config** 変換を使用します。この特定のケースでは、新しい 「**web.release.config**」 エントリを追加します。 

1.  Visual Studio の 「**ソリューション エクスプローラー**」 ペインで 「**Web.config**」 エントリを拡張して 「**Web.Release.config**」 を選択します。Visual Studio ウィンドウの中央のペインで **Web.Release.config** ファイルが開きます。
1.  「**Web.Release.config**」 ペインで、`<configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">` ラインのすぐ下に次のエントリを追加します (`<server_name>` プレースホルダーは、この演習の最初のタスクで作成したサーバーの名前に置き換えます):

    ```csharp
    <connectionStrings>
      <add name="DefaultConnection" connectionString="Data Source=<server_name>.database.windows.net;Initial Catalog=nerddinnerlab;Integrated Security=False;User ID=sqluser;Password=Pa55w.rd1234;Connect Timeout=30;Encrypt=True;TrustServerCertificate=False;ApplicationIntent=ReadWrite;MultiSubnetFailover=False" providerName="System.Data.SqlClient" xdt:Transform="SetAttributes" xdt:Locator="Match(name)"/>
    </connectionStrings>
    ```

    > **注**: アプリケーションの LocalDB が Azure SQL データベースに移行され、接続文字列がデータベースの場所の変更を反映するよう更新されました。

#### タスク 3: Docker サポートを追加し、Visual Studio を使用して Docker コンテナーないでアプリケーションをローカルにデバグする

このタスクでは、Docker サポートを Visual Studio に追加し、これを使って Docker コンテナー内でローカルにアプリケーションのデバッグを行います。

1.  Visual Studio を使用して Docker でアプリケーションをコンテナー化するには、Visual Studio の 「**ソリューション エクスプローラー**」 ウィンドウで 「**NerdDinner**」 プロジェクトを右クリックします。右クリック メニューで 「**追加**」 を選択し、カスケード メニューで 「**コンテナー オーケストレーター サポート**」 をクリックします。
1.  「**コンテナー オーケストレーション サポートの追加**」 ダイアログ ボックスの 「**コンテナー オーケストレーター**」 ドロップダウン リストで、「**Docker Compose**」 を選択して 「**OK**」 をクリックします。Visual Studio ウィンドウの中央のペインで 「**Dockerfile**」 タブが自動的に開きます。
1.  Docker Desktop を起動するよう指示されたら「**はい**」をクリックします。 

    > **注**: Visual Studio は自動的に必要なファイルをソリューションに追加します (**docker-compose** プロジェクトや **Dockerfile** など)。また、プロジェクトを調べて、プロジェクトでの使用に適したベース イメージを判定します。Nerd Dinner ソリューションの場合は、**microsoft/aspnet:4.8-windowsservercore-ltsc2019 ベース イメージを使用するよう選択されています。

    ```csharp
    FROM mcr.microsoft.com/dotnet/framework/aspnet:4.8-windowsservercore-ltsc2019
    ARG source
    WORKDIR /inetpub/wwwroot
    COPY ${source:-obj/Docker/publish} .
    ```
    > **注**: アプリケーションをローカルで実行し、Visual Studio を使用して Docker コンテナー内でデバッグを行い、Azure SQL データベースへの接続をテストするには、**docker-compose** を起動プロジェクトとして設定し、起動します。

1.  Visual Studio の 「**ソリューション エクスプローラー**」 ウィンドウで 「**docker-compose**」 プロジェクトを右クリックし、右クリック メニューで 「**スタートアップ プロジェクトに設定**」 を選択します。
1.  Visual Studio のトップ レベル メニューで 「**Docker Compose**」 をクリックします。

    > **注**: Visual Studio はベース イメージをダウンロードし、その後、デプロイ用のイメージをビルドします。ビルドが完了したら、ローカル ブラウザーを起動するアプリケーションが表示されます。

    > **注**: 利用可能な帯域幅によっては、ダウンロードに長時間かかる可能性があります。

1.  ラボのコンピューターで管理者として**コマンド プロンプト**を起動し、「**管理者: C:\\windows\\system32\cmd.exe**」 から以下を実行して、ローカル docker イメージを一覧表示し、リストに **Dockerfile** で参照されたイメージが含まれていることを確認します:

    ```cmd
    docker images
    ```

#### タスク 4: Docker イメージを Azure コンテナー レジストリに発行する

このタスクでは、Visual Studio を使用して、前のステップでビルドした Docker イメージを Azure コンテナー レジストリに発行します。

1.  Visual Studio の 「**ソリューション エクスプローラー**」 ウィンドウで 「**NerdDinner**」 プロジェクトを右クリックし、右クリック メニューで 「**発行**」 をクリックします。「**発行**」 ウィザードが起動します。
1.  「**発行**」 ウィザードの 「**今日はどこで発行していますか?**」 ペインで、「**Azure**」 オプションが選択されていることを確認し、「**次へ**」 をクリックします。
1.  「**発行**」 ウィザードの 「**どの Azure サービスを使用してアプリケーションをホストしますか?**」 ページで 「**Azure Container Registry**」 オプションを選択し、「**次へ**」 をクリックします。
1.  「**発行**」 ウィザードの 「**既存の Azure Container Registry を選択するか、新しい Azure Container Registry を作成**」 ページで、必要に応じて 「**サインイン**」 をクリックします。指示されたら、このラボで使用している Azure サブスクリプションで少なくとも共同作成者のロールがあるユーザー アカウントを使ってサインインします。
1.  「**発行**」 ウィザードの 「**既存の Azure Container Registry を選択するか、新しい Azure Container Registry を作成**」 ページで、プラス記号をクリックします。
1.  「**新規作成**」 ダイアログ ボックスで以下の設定を指定し、「**作成**」 をクリックします:

    | 設定 | 値 | 
    | --- | --- |
    | DNS プレフィックス | 既定値を受け入れる |
    | 定期売買 | お使いの Azure サブスクリプションの名前 |
    | リソース グループ | **az400m15l01a-RG** |
    | SKU | **Standard** |
    | レジストリの場所 | Azure SQL データベースのデプロイで選択したものと同じ Azure リージョン |

1.  「**発行**」 ウィザードの 「**既存の Azure Container Registry を選択するか、新しい Azure Container Registry を作成**」 ページに戻り、**終了** をクリックします。
1.  Visual Studio インターフェイスの 「**NerdDinner**」 タブで 「**発行**」 をクリックします。

    > **注**: 発行操作が完了するまでお待ちください。発行アクションの結果、Docker イメージが作成され、Azure Container Registry にプッシュされます。

1.  発行に成功したら、Azure portal を表示している Web ブラウザー ウィンドウに切り替えます。「**コンテナー レジストリ**」 を検索して選択し、「**Azure Container Registry**」 で、新しく作成された Azure コンテナー レジストリを示しているエントリをクリックします。そのブレードの左側にある垂直メニューの 「**サービス**」 セクションで 「**リポジトリ**」 をクリックし、「**nerddiner**」 エントリが含まれていることを確認します。

#### タスク 5: 新しい Docker イメージを ACR から Azure Container Instances (ACI) にプッシュする

このタスクでは、Azure Container インスタンス (ACI) を作成し、ACR から新しくアップロードされた Docker イメージにプッシュします。

> **注**: コンテナーを Azure Kubernetes Service (AKS) または Service Fabric にデプロイすることもできます。

> **注**: Azure CLI を使用して Azure Container インスタンスを作成します。

1.  Azure portal のツールバーで、検索テキスト ボックスのすぐ右側にある **Cloud Shell** アイコンをクリックします。 
1.  **Bash** または **PowerShell** のいずれかを選択するためのプロンプトが表示されたら、**「Bash」** を選択します。 

    > **注**: **Cloud Shell** を初めて起動し、「**ストレージがマウントされていません**」というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、「**ストレージの作成**」を選択します。 

1.  「Cloud Shell」 ペインのバッシュ セッションから以下を実行して、ACI 用の新しいリソース グループを作成します:

    ```bash
    RESOURCEGROUPNAME1='az400m15l01a-RG'
    LOCATION=$(az group show --name $RESOURCEGROUPNAME1 --query location --output tsv)
    RESOURCEGROUPNAME2='az400m15l02a-RG'
    az group create --name $RESOURCEGROUPNAME2 --location $LOCATION
    ```

1.  Cloud Shell ペインのバッシュ セッションから以下のコマンドを実行して ACI を作成し、前のタスクで作成した ACR からその ACI に NerdDinner イメージをプッシュします:

    ```bash
    ACINAME='nerddinner'$RANDOM$RANDOM
    ACRNAME=$(az acr list --resource-group $RESOURCEGROUPNAME1 --query '[].name' --output tsv)
    ```

1.  Cloud Shell ペインのバッシュ セッションから以下のコマンドを実行し、前のタスクで作成した ACR への 「**プル**」 許可を使用してサービス プリンシパルを作成し、そのパスワードをシェル変数に格納します:

    ```bash
    SPPASSWORD=$(az ad sp create-for-rbac \
    --name http://$ACRNAME-pull \
    --scopes $(az acr show --name $ACRNAME --query id --output tsv) \
    --role acrpull \
    --query password \
    --output tsv)
    ```

1.  Cloud Shell ペインでバッシュ セッションから以下のコマンドを実行し、前の手順で作成したサービス プリンシパルの名前を取得して、そのパスワードをシェル変数に格納します:

    ```bash  
    SPUSERNAME=$(az ad sp show --id http://$ACRNAME-pull --query appId --output tsv)
    ```

1.  Cloud Shell ペインでバッシュ セッションから以下のコマンドを実行し、イメージが含まれている Azure Container Registry の名前を取得して、これをシェル変数に格納します:

    ```bash  
    ACRLOGINSERVER=$(az acr show --name $ACRNAME --resource-group $RESOURCEGROUPNAME1 --query "loginServer" --output tsv)
    ```

1.  Cloud Shell ペインでバッシュ セッションから以下のコマンドを実行し、Azure Container レジストリに格納されているイメージに基づいて Azure Container インスタンスを作成します。このタスクで作成したプル許可のあるサービス プリンシパルを使用すると、これにアクセスできます。

    ```bash
    az container create \
    --name $ACINAME \
    --resource-group $RESOURCEGROUPNAME2 \
    --image $ACRLOGINSERVER/nerddinner:latest \
    --registry-login-server $ACRLOGINSERVER \
    --registry-username $SPUSERNAME \
    --registry-password $SPPASSWORD \
    --dns-name-label $ACINAME \
    --os-type windows \
    --query ipAddress.fqdn
    ```

    > **注**: デプロイが完了するまで待ちます。5 分間程度かかる場合があります。出力には、Azure Container インスタンスに割り当てられた完全修飾 DNS ドメイン名が含まれます。

1.  ラボのコンピューターで別の Web ブラウザー タブを開き、Azure Container インスタンスをプロビジョニングしたコマンドの出力に基づいて識別した IP アドレスに移動し、NerdDinner アプリケーションが実行中であることを確認します。

## 演習 2: Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。 

> **注**: 新しく作成した Azure リソースのうち、使用しないリソースは必ず削除してください。使用しないリソースを削除しないと、予期しないコストが発生する場合があります。

#### タスク 1: Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。 

1.  Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```sh
    az group list --query "[?starts_with(name,'az400m15l0')].name" --output tsv
    ```

1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを削除します。

    ```sh
    az group list --query "[?starts_with(name,'az400m15l0')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    > **注**: コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

## レビュー

このラボでは、Azure クラウドおよび Windows コンテナーを使用し、コードと構成の変更を最低限に抑えて既存の .NET アプリケーションをモダン化する方法を学習しました。
