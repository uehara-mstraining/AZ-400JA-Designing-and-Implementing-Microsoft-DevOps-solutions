---
lab:
    title: 'ラボ: Azure で Puppet を使用してアプリをデプロイする'
    az400Module: 'モジュール 14: Azure で利用可能なコード ツールとしてのサード パーティ インフラストラクチャ'
---

# ラボ: Azure で Puppet を使用してアプリをデプロイする
# 学生用ラボ マニュアル

## ラボの概要

このラボでは、Puppet Labs の Puppet を使用して PartsUnlimted MRP アプリケーション (PU MRP アプリ) を Azure VM にデプロイします。 

Puppet は、インフラストラクチャの状態をコードとして記述することにより、マシンのプロビジョニングと構成を自動化できる構成管理システムです。 

ラボは以下の高レベルの手順で構成されます。

- Azure Resource Manager テンプレートを使用して Azure VM で Puppet マスターとノードのプロビジョニングを行う
- ノード上に Puppet エージェントをインストールする
- Puppet 運用環境を構成する
- 運用環境の構成をテストする
- Puppet プログラムを作成して PU MRP アプリの前提条件を記述する
- ノード上で Puppet 構成を実行する

## ラボ環境

ラボ環境は 2 つの Azure VM とラボのコンピューターで構成されます。Azure VM には以下が含まれます:

- PU MRP アプリをホストする Puppet ノード。ノードで実行する唯一のタスクは、Puppet エージェントをインストールすることです。Puppet エージェントは Linux または Windows 上で実行できます。このラボでは、Ubuntu サーバーを使用します。
- Puppet マスターは、Puppet プログラムを介してノードに適用された構成を管理します。Puppet マスターは Linux 上で実行する必要があります。このラボでは、Ubuntu サーバーも使用します。

Azure Resource Manager テンプレートを使用して、2 つの Azure VM をデプロイします。ラボのコンピューターは、Puppet マスターとの接続や管理に使用する管理ワークステーションとして機能します。 

## 目標

このラボを完了すると、次のことができるようになります。

- Azure Resource Manager テンプレートを使用して Azure VM で Puppet マスターとノードのプロビジョニングを行う
- ノード上に Puppet エージェントをインストールする
- Puppet 運用環境を構成する
- 運用環境の構成をテストする
- Puppet プログラムを作成する
- ノード上で Puppet 構成を実行する

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

### 演習 1: Puppet を使用して PartsUnlimted MRP アプリケーションを Azure VM にデプロイ

この演習では、Puppet を使用して PartsUnlimted MRP アプリケーションを Azure VM にデプロイします。

#### タスク 1: Puppet マスターおよびそのノードとして機能する Azure VM をプロビジョニングする

このタスクでは、Azure Resource Manager テンプレートを使用して 2 つの Azure VM をデプロイして構成します。最初の VM は Puppet マスターとして機能し、2 つ目はその管理対象ノードになります。

1.  ラボのコンピューターから Web ブラウザーを起動し、[**Azure Portal**](https://portal.azure.com) に移動します。このラボで使用している Azure サブスクリプションで少なくとも共同作成者のロールがあるユーザー アカウントを使ってサインインします。
1.  別のブラウザー タブを開き、[Azure Resource Manager テンプレートのリンク](
https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FPartsUnlimitedMRP%2Fmaster%2FLabfiles%2FAZ-400T05-ImplemntgAppInfra%2FLabfiles%2FM04%2FPuppet%2Fenv%2FPuppetPartsUnlimitedMRP.json) に移動します。これで自動的に Azure portal の 「**カスタム デプロイ**」 ブレードにリダイレクトされます。

1.  Azure portal の 「**カスタム デプロイ**」 ブレードで、「**テンプレートの編集**」 をクリックします
1.  「**テンプレートの編集**」 ブレードのテンプレート概要ペインで 「「**変数**」 セクションを拡張し、「**vmSize**」 をクリックします。
1.  テンプレートの詳細セクションで `"pmVmSize": "Standard_D2_V2",` を `"vmSize": "Standard_D2s_v3",` に変更します。
1.  テンプレートの詳細セクションで、`"mrpVmSize": "Standard_A2",` を `"vmSize": "Standard_D2s_v3",` に変更して 「**保存**」 をクリックします。
1.  再び 「**カスタム デプロイ**」 ブレードで以下の設定を指定します (他は既定値のままにします):

    | 設定 | 値 |
    | --- | --- |
    | 定期売買 | このラボで使用する Azure サブスクリプションの名前 |
    | リソース グループ | 新しいリソース グループ「**az400m14l03a-RG**」の名前 |
    | リージョン | Azure VM をデプロイできる Azure リージョンの名前 |
    | Pm 管理者ユーザー名 | **azureuser** |
    | Pm 管理者パスワード | **Pa55w.rd1234** |
    | パブリック IP の Pm Dns 名 | 文字で始まり、3 ～ 63 字の文字と数字で構成された一意の文字列 |
    | Pm コンソール パスワード | **Pa55w.rd1234** |
    | Mrp 管理者ユーザー名 | **azureuser** |
    | Mrp 管理者パスワード | **Pa55w.rd1234** |
    | パブリック IP の Mrp Dns 名 | 文字で始まり、3 ～ 63 字の文字と数字で構成された一意の文字列 |

    > **注**: デプロイ中に指定した DNS 名を記録しておいてください。最初の名前 (**Pm**) は Puppet マスターに割り当てられます。2 番目 (**Mrp**) は Puppet ノードに割り当てられます。 

1.  「**カスタム デプロイ**」 ブレードで 「**レビューと作成**」 をクリックします。検証プロセスが完了していることを確認して、「**作成**」 をクリックします。

    > **注**: デプロイが完了するのを待ちます。これにはおよそ 10 分かかります。

1.  デプロイの完了後、Azure portal でページの最上部にある 「**リソース、サービス、ドキュメントの検索**」 テキスト ボックスを使い、**仮想マシン** リソースを検索して選択します。「**仮想マシン**」 ブレードで、新しくデプロイされ、Puppet ノードとして使用される Azure VM を示すエントリを選択します。
1.  仮想マシン ブレードで、「**DNS 名**」 プロパティの値を特定して記録します。
1.  「**仮想マシン**」 ブレードに戻り、新しくデプロイされ、Puppet マスターをホストする Azure VM を示すエントリを選択します。
1.  仮想マシンブレードで、マウスのポインターを 「**DNS 名**」 エントリの上に動かし、これをクリップボードにコピーします。 
1.  新しいブラウザー タブを開き、クリップボードにコピーした DNS 名に移動します。**Https** プレフィックスを使用します。これにより、Puppet マスター コンソールのサインイン ページが表示されます

    > **注**: 証明書エラーは無視してください。これは予測されていることです。 

1.  Puppet マスター コンソールのサインイン ページが表示されている Web ブラウザー ウィンドウで、ユーザー名「**admin**」、パスワード「**Pa55w.rd1234**」を使用してサインインします。これにより Puppet 構成管理コンソールが表示されます。

    > **注**: デプロイ中に指定したユーザー名で Puppet マスター コンソールにログインすることはできません。組み込み管理者ユーザー アカウントを使用する必要があります。

#### タスク 2: ノード Azure VM に Puppet エージェントをインストールする

このタスクでは、Puppet マスターが管理するノードとして 2 つ目の Azure VM を追加します。 

1.  Puppet 構成管理コンソールが表示されている Web ブラウザーの左側にある垂直メニューで、「**ノード**」 をクリックします。その後、「**未署名の証明書**」 をクリックし、Puppet Enterprise で管理されるノードを追加できるコマンドを記録します。 

    > **注**: このコマンドは、このタスクで後ほど必要になります。コマンドは以下の形式に類似しています (`<Pm_Dns_Name_for_Public_IP>` プレースホルダーは、前のタスクで 2 つ目の Azure VM の IP アドレスに割り当てた名前を示します):

    ```bash
    curl -k https://<Pm_Dns_Name_for_Public_IP>.0h03b1jj0ewetnu40a35rbm0qg.bx.internal.cloudapp.net:8140/packages/current/install.bash | sudo bash
    ``` 

1.  ラボのコンピューターの Web ブラウザー ウィンドウで [PuTTY ダウンロード ページ](https://putty.org/) に移動し、PuTTY インストーラーをダウンロードします。既定の設定を使ってインストールを実行します。
1.  インストール後、「**スタート**」 メニューで **PuTTY (64-bit)** フォルダーを拡張し、「**PuTTY**」 アイコンをクリックして 「**PuTTY 構成**」 ウィンドウを開きます。
1.  「**PuTTY 構成**」 ウィンドウで 「**ホスト名 (または IP アドレス)**」 テキストボックスに、前のタスクの最後に識別した Puppet ノードをホストする Azure VM の DSN 名を入力し、「**開く**」 をクリックします。
1.  指示されたら、「**PuTTY セキュリティ アラート**」 ウィンドウで 「**はい**」 をクリックします。
1.  指示されたら、Puppet ノードへの 「**PuTTY**」 セッションで、**azureuser** としてサインインします (パスワードは **Pa55w.rd1234**)。
1.  サインイン後、Puppet ノードへの PuTTY セッションで、このタスクで以前に記録しておいたコマンドを実行します。

    > **注**: コマンドが Puppet エージェントとノードの依存関係をインストールするのを待ってください。2 分間程度かかる場合があります。この後は、Puppet マスターのみを使用してノードを排他的に構成します。

    > **注**: Azure Marketplace から Puppet エージェントの拡張機能を使用すると、Azure VM で Puppet エージェントのインストールと構成を自動化できます。

    > **注**: 次に、新しくインストールされたノードを管理するため、Puppet 構成管理コンソールから保留中の要求を承諾する必要があります。

1.  Puppet 構成管理コンソールを表示している Web ブラウザー ウィンドウに戻り、「**未署名の証明書**」 ページを更新します。ページに保留中の未署名証明書の要求が表示されていることを確認して 「**承諾**」 をクリックし、ノードをインベントリに移動する要求を承認します。

    > **注**: これは、Puppet マスターとノード間の通信を確保する証明書を許可する要求です。

1.  Puppet 構成管理コンソールを表示している Web ブラウザー ウィンドウで 「**ノード**」 をクリックし、それぞれ Puppet マスターとノードを示す 2 つのエントリが表示されていることを確認します。

    > **注**: Parts Unlimited MRP アプリケーション (PU MRP App) は Java アプリケーションです。PU MRP アプリでは、Puppet ノードとして構成された Azure VM で MongoDB と Apache Tomcat をインストールして構成する必要があります。ただし、MongoDB と Tomcat を手動でインストールして構成する代わりに、自動的にノードを構成する Puppet プログラムを書き込みます。Puppet プログラムは、Puppet マスターで特定のディレクトリに格納されます。Puppet プログラムは、ターゲット ノードの希望する状態を説明するマニフェストで構成されます。マニフェストはモジュール (事前にパッケージ化された Puppet プログラム) を使用します。ユーザーは独自のモジュールを作成するか、Puppet Labs が維持している「Forge」という市販のモジュールを使用できます。Forge の一部のモジュールは正式にサポートされていますが、コミュニティからアップロードされるオープンソースのモジュールもあります。Puppet プログラムは環境によって決まるため、開発、テスト、運用といったさまざまな環境で Puppet プログラムを管理できます。

    > **注**: このラボでは、新しく準備が完了したノードを使用して、運用環境をエミュレートします。また、Forge からモジュールをダウンロードし、ノードを構成するために使用します。

#### タスク 3: Puppet 運用環境を構成する

このタスクでは、エミュレートされた Puppet 運用環境を構成します。 

> **注**: Puppet を Azure にデプロイするために使用したテンプレートでは、運用環境を管理するために Puppet マスターでディレクトリも構成されました。該当するディレクトリは、**etc/puppetlabs/code/environments/production** です。

> **注**: 最初に運用モジュールを点検します。

1.  ラボのコンピューターの 「**スタート**」 メニューで **PuTTY (64-bit)** フォルダーを拡張し、「**PuTTY**」 アイコンをクリックして 「**PuTTY 構成**」 ウィンドウを開きます。
1.  「**PuTTY 構成**」 ウィンドウで 「**ホスト名 (または IP アドレス)**」 テキストボックスに、Puppet マスターをホストする Azure VM の DSN 名を入力し、「**開く**」 をクリックします。
1.  指示されたら、「**PuTTY セキュリティ アラート**」 ウィンドウで 「**はい**」 をクリックします。
1.  指示されたら、Puppet マスターへの 「**PuTTY**」 セッションで、**azureuser** としてサインインします (パスワードは **Pa55w.rd1234**)。
1.  承認後、Puppet マスターへの PuTTY セッションで以下を実行し、現在の作業ディレクトリを運用ディレクトリ (**etc/puppetlabs/code/environments/production**) に変更します:

    ```bash
    cd /etc/puppetlabs/code/environments/production
    ```

1.  Puppet マスターへの PuTTY セッションから `ls` コマンドを実行し、運用ディレクトリのコンテンツを一覧表示します。 

    > **注**: 「**manifests**」と「**modules**」という名前のディレクトリが表示されます。「**manifests**」ディレクトリには、このラボで後ほどサンプル ノードに適用する構成の説明が含まれています。「**modules**」ディレクトリには、マニフェストが参照するモジュールが含まれています。

    > **注**: 次に、サンプル ノードの構成に必要な追加の Puppet モジュールを Forge からインストールします。 

1.  Puppet マスターへの PuTTY セッション内で、以下のコマンドを実行して必要なモジュールをインストールします。

    ```bash
    sudo puppet module install puppetlabs-mongodb
    sudo puppet module install puppetlabs-tomcat
    sudo puppet module install maestrodev-wget
    sudo puppet module install puppetlabs-accounts
    sudo puppet module install puppetlabs-java
    ```

    > **注**: Forge からの mongodb と tomcat モジュールは正式にサポートされています。Wget モジュールはユーザー モジュールで、正式にサポートされていません。アカウント モジュールは、Linux でユーザーとグループを管理・作成するためのクラスを Puppet に提供します。最後に、java モジュールはさらなる Java 機能を Puppet に提供します。

    > **注**: 次に、Puppet マスターの **modules** ディレクトリで「**mrpapp***」という名前のカスタム モジュールを作成します。このカスタム モジュールは PU MRP アプリを構成します。 

1.  Puppet マスターへの PuTTY セッションから以下のコマンドを実行して、現在のディレクトリを **etc/puppetlabs/code/environments/production/modules** に変更します:

    ```bash
    cd /etc/puppetlabs/code/environments/production/modules
    ```

1.  Puppet マスターへの PuTTY セッションから以下のコマンドを実行して、**mrpapp** モジュールを作成します:

    ```bash
    sudo puppet module generate partsunlimited-mrpapp
    ```

    > **注**: これによりウィザードが起動し、モジュールのスキャフォールディングを行いながら一連の質問をします。 

1.  **Mrapp** モジュールの作成を完了するには、ウィザードが終わるまで質問ごとに **Enter** キーを押します (空白または既定を承諾)。

    > **注**: `ls -la` コマンドを使用し、新しいモジュールが作成されたことを確認します。`ls -la` コマンドは、長いリスト形式 (`-l`) を使用してディレクトリの内容を一覧表示します (隠しファイル (`-a`) も含まれます)。

    > **注**: Mrpapp モジュールはノードの構成を定義します。運用環境でのノードの構成は、**site.pp** ファイルで定義されます。**Site.pp** ファイルは **manifests** ディレクトリにあります。**pp** というファイル名の拡張子は、「**Puppet Program**」の頭字語です。ノードの構成を追加して、**site.pp** ファイルを編集します。

1.  Puppet マスターへの PuTTY セッションから以下のコマンドを実行して、Nano テキスト エディターで **site.pp** を開きます。

    ```bash
    sudo nano /etc/puppetlabs/code/environments/production/manifests/site.pp
    ```

1.  Nano エディター インターフェイス内でファイルの最下部までスクロールし、既存の `node default` セクションを以下に置き換えます:

    ```bash
    node default {
       class { 'mrpapp': }
    }
    ```

1.  Nano エディター インターフェイス内で **ctrl + o** キーの組み合わせを押し、**Enter** キーを押してから **ctrl + x** キーの組み合わせを押して変更を保存し、ファイルを閉じます。

    > **注**: **site.pp** ファイルを編集することにより、既定で **mrpapp** モジュールですべてのノードを構成するよう Puppet に指示します。**mrpapp** モジュール (現在は空) は運用環境の **modules** ディレクトリにあるので、Puppet はモジュールの場所を把握できます。

#### タスク 4: 運用環境の構成をテストする 

このタスクでは、運用環境の構成をテストします 

> **注**: サンプル ノード向けに PU MRP アプリを完全に記述する前に、**mrpapp** モジュールでダミー ファイルを設定して構成をテストします。Puppet がダミー ファイルを実行して作成したら、**mrpapp** モジュールの完全な構成で続行できます。

> **注**: まず、**init.pp** ファイルを編集します。これは **mrpapp** モジュールのエントリ ポイントになります。このファイルは、Puppet マスターの **mrpapp/manifests** ディレクトリにあります。 

1.  Puppet マスターへの PuTTY セッションから以下のコマンドを実行し、Nano テキスト　エディターで **init.pp** ファイルを開きます。

    ```bash
    sudo nano /etc/puppetlabs/code/environments/production/modules/mrpapp/manifests/init.pp
    ```

1.  Nano エディター インターフェイス内で `class: mrpapp` 宣言までスクロールし、これを変更して以下のようにします:

    ```puppet
    class mrpapp {
       file { '/tmp/dummy.txt':
          ensure => 'present',
          content => 'Puppet rules!'
       }
    }
    ```

    > **注**: 指示されているように、クラスの定義の前にあるコメント タグ (`#`) を必ず削除してください。

1.  Nano エディター インターフェイス内で **ctrl + o** キーの組み合わせを押し、**Enter** キーを押してから **ctrl + x** キーの組み合わせを押して変更を保存し、ファイルを閉じます。

    > **注**: Puppet プログラムのクラスは、オブジェクト指向プログラミングのクラスとは異なります。Puppet プログラムのクラスは、ノードで構成されるリソースを定義します。`mrpapp` クラスまたはリソースを **init.pp** ファイルに追加することにより、ファイルが「/tmp/dummy.txt」パスにあり、ファイルに「Puppet rules!」の内容が含まれていることを確認するよう Puppet に指示しました。このラボが進むに従い、`mrpapp` クラス内では、より高度なリソースを定義します。

    > **注**: 次に、新しく設定されたテストを実行します。 

1.  Puppet ノードへの PuTTY セッションに切り替え、以下のコマンドを実行して新しい構成をテストします:

    ```bash
    sudo puppet agent --test --debug
    ```

    > **注**: 既定により、Puppet エージェントは 30 分ごとに Puppet マスターに構成をクエリします。その後、Puppet エージェントは、Puppet マスターの指定した構成に対して現在の構成をテストします。必要に応じて、Puppet エージェントは構成を変更し、Puppet マスターの指定した構成に一致するようにします。呼び出したコマンドにより、Puppet マスターに対してローカル Puppet エージェントによる即時構成チェックがトリガーされます。この場合、構成には **tmp/dummy.txt** ファイルが必要です。これに従ってノードはファイルを作成します。

1.  Puppet ノードへの PuTTY セッション内で以下のコマンドを実行し、**tmp/dummy.txt** ファイルの存在を確認して、ファイルの内容を一覧表示します:

    ```bash
    cat /tmp/dummy.txt
    ```

1.  「Puppet rules!」メッセージがコマンドの出力として表示されていることを確認します。 

    > **注**: 次に、構成ドリフトを修正します。Puppet エージェントを実行するたびに、Puppet は環境が適正な状態かどうか判断します。適正な状態でない場合、Puppet は必要に応じて関連のあるクラスを再適用します。このプロセスにより、Puppet は構成ドリフトを検出して修正できます。ダミー ファイル「**dummy.txt**」をノードから削除して、構成ドリフトをシミュレートします。 

1.  Puppet ノードへの PuTTY セッション内で以下を実行して **dummy.txt** ファイルを削除します:

    ```bash
    sudo rm /tmp/dummy.txt
    ```

1.  Puppet ノードへの PuTTY セッション内で以下のコマンドを実行し、Puppet エージェントの実行をトリガーします:

    ```bash
    sudo puppet agent --test
    ```

1.  Puppet ノードへの PuTTY セッション内で以下のコマンドを実行し、**tmp/dummy.txt** ファイルの存在を確認して、ファイルの内容を一覧表示します:

    ```bash
    cat /tmp/dummy.txt
    ```

#### タスク 5: Puppet プログラムを作成して PU MRP アプリの前提条件を記述する

このタスクでは、Puppet プログラムを作成して PU MRP アプリの前提条件を記述します。

> **注**: Puppet マスターにノードがフックされました。これで Puppet プログラムを書き込み、PU MRP アプリの前提条件を記述できます。実際には、大きな構成ソリューションのさまざまな部分が通常、複数のマニフェストやモジュールに分割されます。複数のファイルで構成を分割することは一種のモジュラー化で、コードの再使用が促進されます。作業を簡単にするため、このラボでは、以前に作成した mrpapp モジュール内で構成全体を単一の Puppet プログラム ファイル「**init.pp**」に記述します。このタスクでは、順を追って **init.pp** ファイルを構築します。

> **注**: 完了した **init.pp** ファイルは、[Microsoft の PU MRP GitHub リポジトリ](https://raw.githubusercontent.com/Microsoft/PartsUnlimitedMRP/master/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/M04/Puppet/final/init.pp) で見つけられます。**Init.pp** ファイルの内容を GitHub からコピーするか、このタスクで順を追って編集します。

> **注**: Tomcat では、Tomcat サービスにアクセスして実行するため、ノード上に Linux ユーバーとグループが必要です。Tomcat が両方で使用している既定の名前は `Tomcat` です。この校正を実装する方法はいくつかありますが、ここでは **init.pp** ファイルの別個のクラスを使用します。**Init.pp** ファイルの編集に加えて、以下の変更を行います。

- **War.pp** ファイルを編集し、PU MRP アプリのデプロイを簡素化します。
- **~/tomcat7/webapps** ディレクトリでアクセス許可を編集し、このタスクの最後に **war** ファイルを抽出できるようにします。このアクセス許可を編集すると、**pp** ファイルの実行時に Tomcat サービスを再起動した際、**war** ファイルを自動的に抽出できるようになります。

> **注**: Puppet プログラムの作成に必要なアクションすべてについては、このタスクの各セクションを経る際に説明します。 

##### セクション 5.1: MongoDB の構成

> **注**: MongoDB の構成後、Puppet が PU MRP アプリケーションのデータベース向けデータが含まれている Mongo スクリプトをダウンロードするようにします。これは、MongoDB セットアップの一部として含めます。このような要件を実装するには、クラスを追加して MongoDB を構成します。

1.  Puppet マスターへの PuTTY セッションで以下のコマンドを実行し、Nano テキスト エディターの **mrpapp module** 内で **init.pp** ファイルを開きます。

    ```bash
    sudo nano /etc/puppetlabs/code/environments/production/modules/mrpapp/manifests/init.pp
    ```

1.  Nano エディター インターフェイス内でファイルの最下部までスクロールし、新しいラインから始めて `configuremongodb` を追加します:

    ```puppet
    class configuremongodb {
      include wget
      class { 'mongodb': }->

      wget::fetch { 'mongorecords':
        source => 'https://raw.githubusercontent.com/Microsoft/PartsUnlimitedMRP/master/deploy/MongoRecords.js',
        destination => '/tmp/MongoRecords.js',
        timeout => 0,
      }->
      exec { 'insertrecords':
        command => 'mongo ordering /tmp/MongoRecords.js',
        path => '/usr/bin:/usr/sbin',
        unless => 'test -f /tmp/initcomplete'
      }->
      file { '/tmp/initcomplete':
        ensure => 'present',
      }
    }
    ```

1.  Nano エディター インターフェイス内で **ctrl + o** キーの組み合わせを押してから **Enter** キーを押し、変更を保存しますが、ファイルは開いたままにしておきます。 

    > **注**: `configuremongodb` クラスを検証してみましょう。

    ライン 1`configuremongodb` という `class (resource)` を作成します。
    ライン 2`wget` モジュールを含めたので、`wget` を使ってファイルをダウンロードできます。
    ライン 3`mongodb resource` を呼び出します (`mongodb` モジュールから以前にダウンロードしていたもの)。これにより、Puppet Mongo DB モジュールで定義した既定値を使用して Mongo DB がインストールされます。これで Mondo DB をインストールできます。
    ライン 5`Fetch` リソースを `wget` モジュールから起動し、`mongorecords` リソースを呼び出します。
    ライン 6PU MRP GitHub からダウンロードする必要があるファイルのソースを設定します。
    ライン 7ファイルのダウンロード先を設定します。
    ライン 10組み込み Puppet リソース `exec` を使用してコマンドを実行します。
    ライン 11実行するコマンドを指定します。
    ライン 12コマンド呼び出しのパスを設定します。
    ライン 13`unless` というキーワードを使用して条件を指定します。このコマンドは一度だけ実行し、記録の挿入後に `tmp` ファイルを作成するようにします (ライン 15)。このファイルがある場合は、コマンドを再び実行しません。

    > **注**: ライン 3、9、14 の `->` 表記は順序付けの矢印です。順序付けの矢印は、右のリソースを呼び出す前に左のリソースを適用するよう Puppet に指示します。これにより、実行の順序を制御できます。

##### セクション 5.2: Java を構成する

1.  Nano エディター インターフェイス内で **init.pp** ファイルの最後に以下の `configurejava` クラス定義を `configuremongodb` クラス定義のすぐ下に追加して Java 要件を設定します。

    ```puppet
    class configurejava {
      include apt
      $packages = ['openjdk-8-jdk', 'openjdk-8-jre']

      apt::ppa { 'ppa:openjdk-r/ppa': }->
      package { $packages:
         ensure => 'installed',
      }
    }
    ```

1.  Nano エディター インターフェイス内で **ctrl + o** キーの組み合わせを押してから **Enter** キーを押し、変更を保存しますが、ファイルは開いたままにしておきます。 

    > **注**: `configurejava` クラスを検証してみましょう。

    ライン 2`Apt` モジュールを含めます。これにより、新しい **パーソナル パッケージ アーカイブ (PPA)** を構成できます。
    ライン 3インストールする必要のあるパッケージの配列を作成します。
    ライン 5**PPA** を追加します。
    ライン 6 ～ 8。パッケージがインストールされていることを確認するよう Puppet に指示します。Puppet は配列を拡張し、配列の各パッケージをインストールします。

    > **注**: Puppet パッケージ ターゲットを使用して Java をインストールすることはできません。Java 7 しかインストールされないためです。PPA を追加し、`apt` モジュールを使用して Java 8 をインストールします。

##### セクション 5.3: ユーザーとグループを作成する

1.  Nano エディター インターフェイス内で、**init.pp** ファイルの最後に以下の `createuserandgroup` クラスを追加し、サービスの構成、デプロイ、開始、停止で使用する Linux ユーザーとグループを指定します。

    ```puppet
    class createuserandgroup {

    group { 'tomcat':
      ensure => 'present',
      gid    => '10003',
     }

    user { 'tomcat':
      ensure           => 'present',
      gid              => '10003',
      home             => '/tomcat',
      password         => '!',
      password_max_age => '99999',
      password_min_age => '0',
      uid              => '1003',
     }
    }
    ```

1.  Nano エディター インターフェイス内で **ctrl + o** キーの組み合わせを押してから **Enter** キーを押し、変更を保存しますが、ファイルは開いたままにしておきます。 

    > **注**: `createuserandgroup` クラスを検証してみましょう。

    ライン 1**Init.pp** ファイルの最初で定義した `createuserandgroup` クラスを呼び出します。このクラスは、`user` と `group` リソースを使用するよう定義されています。
    ライン 3 ～ 6。`Group` コマンドを使用して `Tomcat` グループを作成します。パラメータ値 `ensure = > present` を使用して、`Tomcat` グループが存在することを確認し、必要であれば作成します。その後、`gid` 値をこのグループに割り当てます。この割り当ては必須ではありませんが、複数のマシンでグループを管理する際に役立ちます。
    ライン 8 ～ 17。`User` コマンドを使用して、ユーザー `Tomcat` を作成します。`Ensure` コマンドを使用して、`Tomcat` ユーザーが存在することを確認し、必要であれば作成します。ユーザーを管理できるように `gid` および `uid` 値を割り当てます。その後、`Tomcat` ユーザーのホーム ディレクトリ パスを設定し、ユーザーのパスワード設定を割り当てます。使いやすいように `password` 値は未指定のままにしてあります。実際には厳格なパスワード ポリシーを実装すべきです。

    > **注**: `user` および `group` コマンドは `puppetlabs-accounts` モジュールを介して利用できます。これはラボで以前に Puppet マスターにインストールされたものです。

##### セクション 5.4: Tomcat を構成する

1.  Nano エディター インターフェイス内で **init.pp** ファイルの最後に以下の `configuretomcat` クラスを追加して Tomcat を構成します:

    ```puppet
    class configuretomcat {
      class { 'tomcat': }
      require createuserandgroup

     tomcat::instance { 'default':
      catalina_home => '/var/lib/tomcat7',
      install_from_source => false,
      package_name => ['tomcat7','tomcat7-admin'],
     }->

     tomcat::config::server::tomcat_users {
     'tomcat':
       catalina_base => '/var/lib/tomcat7',
       element  => 'user',
       password => 'password',
       roles => ['manager-gui','manager-jmx','manager-script','manager-status'];
     'tomcat7':
       catalina_base => '/var/lib/tomcat7',
       element  => 'user',
       password => 'password',
       roles => ['manager-gui','manager-jmx','manager-script','manager-status'];
     }->

     tomcat::config::server::connector { 'tomcat7-http':
      catalina_base => '/var/lib/tomcat7',
      port => '9080',
      protocol => 'HTTP/1.1',
      connector_ensure => 'present',
      server_config => '/etc/tomcat7/server.xml',
     }->

     tomcat::service { 'default':
      use_jsvc => false,
      use_init => true,
      service_name => 'tomcat7',
     }
    }
    ```

1.  Nano エディター インターフェイス内で **ctrl + o** キーの組み合わせを押してから **Enter** キーを押し、変更を保存しますが、ファイルは開いたままにしておきます。 

    > **注**: `configuretomcat` クラスを検証してみましょう。

    ライン 1`configuretomcat` という `class (resource)` を作成します。
    ライン 2以前にダウンロードしていた Tomcat モジュールから `Tomcat` リソースを呼び出します。
    ライン 3Tomcat を構成する前に `Tomcat` ユーザーとグループを利用できるようにしなくてはならないため、`createuserandgroup` クラスを必須としてマークします。
    ライン 6 ～ 9。以前にダウンロードしていた Tomcat モジュールから `Tomcat` パッケージをインストールします。パッケージ名は `Tomcat7` なので、ここでもそう呼びます。既定のインストールを配置するホーム ディレクトリを指定します。その後、それがソースからでないことを指定します。これは、ダウンロードされたパッケージからインストールする際に必要です。希望すれば、Web ソースから直接インストールできます。また、`tomcat7-admin` パッケージをインストールします。このラボでは `tomcat7-admin` パッケージを使用しませんが、Tomcat Manager Web ユーザー インターフェイスを提供するために使用できます。
    ライン 13 ～ 23。Tomcat で使用できるように 2 つのユーザー名を作成します。これは、Tomcat を管理して構成するために Tomcat アプリケーション内で作成されます。**var/lib/tomcat7/conf/tomcat-users.xml** ファイル内で作成され、ここから呼び出されます。 
    ライン 26 ～ 31。Tomcat コネクタを構成し、使用するプトロコルとポート番号を定義します。また、コネクタがあることを確認し、設定したポートで要求に応えられるようにします。さらに、Puppet が Tomcat **server.xml** ファイルに書き込めるようにコネクタ プロパティを指定します。**var/lib/tomcat7/conf/server.xml** で Puppet マスター上に再びファイルを開き、その内容を表示できます。
    ライン 35 ～ 38。Tomcat サービスを構成します。Tomcat のインストールは 1 件のみなので、構成ターゲットとして `default` を指定します。`Use_init` 値を設定して、サービスが実行中であることを確認します。その後、PU MRP アプリのサービス コネクタ名を `tomcat7` と指定します。

##### セクション 5.5: WAR ファイルをデプロイする

> **注**: PU MRP アプリは、 .war ファイルにコンパイルされ、Tomcat はこれを使用してページに対応します。PU MRP アプリ サイトで .war ファイルをデプロイするためのリソースを指定する必要があります。

1.  Nano エディター インターフェイス内で **init.pp** ファイルの最後に以下の `deploywar` クラスを追加します。

    ```puppet
    class deploywar {
      require configuretomcat

      tomcat::war { 'mrp.war':
        catalina_base => '/var/lib/tomcat7',
        war_source => 'https://raw.githubusercontent.com/Microsoft/PartsUnlimitedMRP/master/builds/mrp.war',
      }

     file { '/var/lib/tomcat7/webapps/':
       path => '/var/lib/tomcat7/webapps/',
       ensure => 'directory',
       recurse => 'true',
       mode => '777',
     }
    }
    ```

1.  Nano エディター インターフェイス内で **ctrl + o** キーの組み合わせを押してから **Enter** キーを押し、変更を保存しますが、ファイルは開いたままにしておきます。 

    > **注**: deploywar クラスを検証してみましょう。

    ライン 1`deploywar` と呼ばれる `class (resource)` を作成します。
    ライン 2`deploywar` クラスを呼び出す前に `configuretomcat` が完了していることを確認するよう Puppet に指示します。
    ライン 5`Catalina_base` ディレクトリを設定し、Puppet が .war を使用中の Tomcat サービスにデプロイするようにします。
    ライン 6Tomcat モジュールの `war` リソースを使用して、.war を `war_source` からデプロイします。
    ライン 9Tomcat モジュールで定義した `file` クラスを呼び出します。`file` クラスでは、使用中のシステムでファイルを構成できます。
    ライン 10PU MRP アプリの Web ディレクトリへのパスを指定します。
    ライン 11構成を適用する前に、指定済みのパスにディレクトリがあることを確認します。
    ライン 12構成を再帰的に適用するよう指定します。これにより、`~/tomcat7/webapps/` ディレクトリ内のすべてのファイルとサブディレクトリに影響するようになります。
    ライン 13`~/tomcat7/webapps/` ディレクトリ内のあらゆるファイルとサブディレクトリで `777` アクセス許可を指定します。このようなアクセス許可では、ターゲット ディレクトリ内のあらゆるコンテンツでアクセス許可を読み取り、書き込み、実行できるように設定することが非常に重要です。このラボでは、`777` アクセス許可を設定できます。常に各ユーザー、グループ、アクセス レベルに従い、より制限的なアクセス許可を設定する必要があります。

    > **注**: 通常、.war ファイルをデプロイするには以下のアクションが必要です

    1. .war ファイルを Tomcat **var/lib/tomcat7/webapps** ディレクトリにコピーします
    2. **webapps** ディレクトリで適正なアクセス許可を設定します
    3. Tomcat サーバー構成ファイル (**var/lib/tomcat7/conf/server.xml**) で `unpackWARS` と `autoDeploy` の値を指定します 
    4. Tomcat ファイル (**var/lib/tomcat7/conf/tomcat-users.xml**) で有効かつ利用可能なユーザーを一覧表示します

    > **注**: Tomcat はサービスの開始時に .war を抽出してからデプロイします。このため、**init.pp** ファイルでサービスを再開します。このラボを完了すると、Tomcat ファイルの **server.xml** と **tomcat-users.xml** を開いて内容を表示できるようになります。

##### セクション 5.6: オーダリング サービスを開始する

> **注**: PU MRP アプリはオーダリング サービスを呼び出します。これは、Mongo DB でオーダーを管理する REST API です。オーダリング サービスは .jar ファイルにコンパイルされます。使用しているノードに .jar ファイルをコピーする必要があります。その後、ノードがバックグラウンドでオーダリング サービスを実行するので、REST API の要求をリッスンできます。

1.  Nano エディター インターフェイス内で **init.pp** ファイルの最後に以下の 'orderingservice` クラスを追加し、オーダリング サービスが実行することを確認します。

    ```puppet
    class orderingservice {
      package { 'openjdk-7-jre':
        ensure => 'installed',
      }

      file { '/opt/mrp':
        ensure => 'directory'
      }->
      wget::fetch { 'orderingsvc':
        source => 'https://raw.githubusercontent.com/Microsoft/PartsUnlimitedMRP/master/builds/ordering-service-0.1.0.jar',
        destination => '/opt/mrp/ordering-service.jar',
        cache_dir => '/var/cache/wget',
        timeout => 0,
      }->
  
      exec { 'stoporderingservice':
        command => "pkill -f ordering-service",
        path => '/bin:/usr/bin:/usr/sbin',
        onlyif => "pgrep -f ordering-service"
      }->

      exec { 'stoptomcat':
        command => 'service tomcat7 stop',
        path => '/bin:/usr/bin:/usr/sbin',
        onlyif => "test -f /etc/init.d/tomcat7",
      }->
      exec { 'orderservice':
        command => 'java -jar /opt/mrp/ordering-service.jar &',
        path => '/usr/bin:/usr/sbin:/usr/lib/jvm/java-8-openjdk-amd64/bin',
      }->
      exec { 'wait':
        command => 'sleep 20',
        path => '/bin',
        notify => Tomcat::Service['default']
      }
    }
    ```

1.  Nano エディター インターフェイス内で **ctrl + o** キーの組み合わせを押してから **Enter** キーを押し、変更を保存しますが、ファイルは開いたままにしておきます。 

    > **注**: orderingservice クラスを検証してみましょう。

    ライン 1`orderingservice` と呼ばれる `class (resource)` を作成します。
    ライン 2 ～ 4。Puppet のパッケージ リソースを使用してアプリケーションの実行に必要な Java Runtime Environment (JRE) をインストールします。
    ライン 6 ～ 8。`/opt/mrp` ディレクトリが存在することを確認し、必要に応じて作成します。
    ライン 9 ～ 14。`wget` を使用してオーダリング サービス バイナリを取得し、`/opt/mrp` に配置します。
    ライン 12キャッシュ ディレクトリを指定し、ファイルが一度だけダウンロードされるようにします。
    ライン 15 ～ 19。実行している場合は orderingservice を停止します。
    ライン 20 ～ 24。実行している場合は tomcat7 サービスを停止します。
    ライン 25 ～ 28。オーダリング サービスを開始します。
    ライン 29 ～ 33。20 秒待ち、オーダリング サービスに起動する時間を与えてから、tomcat サービスに通知します。このようなアクションにより、サービスで更新がトリガーされます。Puppet は、サービス向けに定義した状態を再適用します (すでに実行中でない場合は開始します)。

    > **注**: このサービスは Tomcat を起動する前に実行していなくてはならないため、java コマンドを実行した後に待つ必要があります。さもなければ、オーダリング　サービスが受信 API 要求についてリッスンする必要のあるポートを Tomcat が占有してしまいます。

##### セクション 5.7: mrpapp リソースを完了する

1.  Nano エディター インターフェイス内で **init.pp** ファイルの最上部までスクロールし、このラボで以前にテストした **dummy.txt** ファイルに関連のあるコンテンツを削除して、`mrpapp` クラス定義を削除します。 
1.  Nano エディター インターフェイス内で、更新された `mrpapp` クラスの定義を追加します:

    ```puppet
    class mrpapp {
      class { 'configuremongodb': }
      class { 'configurejava': }
      class { 'createuserandgroup': }
      class { 'configuretomcat': }
      class { 'deploywar': }
      class { 'orderingservice': }
    }
    ```

    > **注**: `mrpapp` クラスの変更により、リソースを実行するために **init.pp** ファイルでどのクラスが必要なのかを指定します。

1.  Nano エディター インターフェイス内で **ctrl + o** キーの組み合わせを押し、**Enter** キーを押してから **ctrl + x** キーの組み合わせを押して変更を保存し、ファイルを閉じます。

    > **注**: **init.pp** ファイルへのあらゆる必要な変更を行うと、[Microsoft の PU MRP GitHub リポジトリ](https://raw.githubusercontent.com/Microsoft/PartsUnlimitedMRP/master/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/M04/Puppet/final/init.pp) の **init.pp** ファイルと同一になるはずです。希望すれば、**init.pp** ファイルの内容を GitHub からコピーし、ローカル **init.pp** ファイルに貼り付けることができます。

##### セクション 5.8: .war ファイル拡張機能のアクセス許可を構成する

> **注**: **init.pp** ファイルで定義した構成の一部では、PU MRP アプリの .war ファイルを **var/lib/tomcat7/webapps** ディレクトリにコピーします。Tomcat は自動的に .war ファイルを webapps ディレクトリから抽出します。ただし、このラボでは、**war.pp** ファイルで完全な読み取り、書き込み、アクセス許可を設定し、ファイルを抽出できるようにします。

1.  Puppet マスターへの PuTTY セッション内で以下のコマンドを実行し、Nano エディターで **war.pp** ファイルを開きます。

    ```bash
    sudo nano /etc/puppetlabs/code/environments/production/modules/tomcat/manifests/war.pp
    ```

1.  Nano エディター インターフェイス内でファイルの最下部にスクロールし、既存の `mode => '0640'` エントリを `mode => '0777'` に置き換えて、意図された許可を変更します。これにより、次のコンテンツが生じます。

    ```puppet
        file { "tomcat::war ${name}":
          ensure    => file,
          path      => "${_deployment_path}/${_war_name}",
          owner     => $user,
          group     => $group,
          mode      => '0777',
          subscribe => Archive["tomcat::war ${name}"],
        }
      }
    }
    ```

1.  Nano エディター インターフェイス内で **ctrl + o** キーの組み合わせを押し、**Enter** キーを押してから **ctrl + x** キーの組み合わせを押して変更を保存し、ファイルを閉じます。

    > **注**: war.pp ファイルのこのセクションを検証してみましょう。

    ライン 1ファイル名を指定します。これは Tomcat モジュール `tomcat::war` に伴います。
    ライン 2アクションを実行する前にファイルがあることを確認します。
    ライン 3Web ディレクトリへのパスを定義します。これは以前、`/var/lib/tomcat7/webapps` に設定したものです。**init.pp** ファイルで指定されているように、ここに PU MRP アプリの .war ファイルがコピーされます。
    ライン 4 ～ 5。ファイルのユーザーとグループの所有者を設定します。
    ライン 6ファイルのアクセス許可モードを `0777` に設定します。これにより、読み取り、書き込み、実行許可が全員に付与されます。運用環境では `777` 許可を使用できません。

#### タスク 6: Puppet ノードで Puppet 構成を実行する

このタスクでは、Puppet ノードで構成の更新をトリガーし、PU MRP アプリのデプロイが成功していることを確認します。

1.  Puppet ノードへの PuTTY セッションに切り替え、以下のコマンドを再実行して新しい構成をテストします:

    ```bash
    sudo puppet agent --test
    ```

    > **注**: 必要なコンポーネントをすべてダウンロードしてインストールする必要があるため、この最初の実行には数分かかります。その後の実行では、Puppet エージェントは既存の環境が適正に構成されていることのみを確認します。 

    > **注**: `openjdk-7-jre` に関するエラー メッセージは無視してください。これは予測されていることです。OpenJDK 6 および 7 は、Ubuntu 16.04 では利用できません。

1.  Tomcat が適正に実行していることを確認するには、Web ブラウザーを起動し、`http:\\` プレフィックスの後に、Puppet ノードをホストする Azure VM の DNS 名と ':9080’ が続く URL に移動します。ブラウザーに Tomcat のランディング ページが表示されていることを確認します。
1.  PU MRP アプリが適正に実行中であることを確認するには、同じブラウザー タブ内で URL に `/mrp` のサフィックスを追加します。ブラウザーに PU MRP アプリの「ようこそ」ページが表示されていることを確認します。

### 演習 3: Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。 

> **注**: 新しく作成した Azure リソースのうち、使用しないリソースは必ず削除してください。使用しないリソースを削除しないと、予期しないコストが発生する場合があります。

#### タスク 1: Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。 

1.  Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```sh
    group list --query "「?starts_with(name,'az400m14l03a')」.name" --output tsv
    ```

1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを削除します。

    ```sh
    az group list --query "「?starts_with(name,'az400m14l03a-RG')」.「name」" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    > **注**: コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

## レビュー

このラボでは、Puppet Labs の Puppet を使用して PartsUnlimted MRP アプリケーション (PU MRP アプリ) を Azure VM にデプロイする方法を学習しました。 
