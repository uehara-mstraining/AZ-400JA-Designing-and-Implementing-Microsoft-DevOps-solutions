---
lab:
    title: 'ラボ: Azure と Ansible'
    module: 'モジュール 14: Azure で利用可能なコード ツールとしてのサード パーティ インフラストラクチャ'
---

# ラボ: Azure と Ansible
# 学生用ラボ マニュアル

## ラボの概要

このラボでは、Ansible を使用して Azure リソースをデプロイ、構成、管理します。 

Ansible は宣言型構成管理ソフトウェアです。プレイブックという形式で管理対象コンピューター向けに意図された構成の説明に依存します。Ansible は自動的にその構成を適用し、その後はこれを維持して、潜在的な不一致に対応します。プレイブックは通常、YAML を使用して書式化されます。

Puppet や Chef といった他の大半の構成管理ツールとは異なり、Ansible はエージェントレスなので、管理対象マシンでソフトウェアをインストールする必要はありません。Ansible は SSH を使用して Linux サーバーを管理し、Powershell Remoting を使用して Windows サーバーとクライアントを管理します。

オペレーティング システム以外のリソース (たとえば、Azure Resource Manager を介してアクセスできる Azure リソース) を操作するため、Ansible はモジュールと呼ばれる拡張機能に対応します。Ansible は Python で書き込まれるため、実質的にモジュールは Python ライブラリとして実装されます。Ansible は [GitHub ホステッド モジュール](https://github.com/ansible-collections/azure) を利用して、Azure リソースを管理します。

Ansible では、管理対象リソースをホスト インベントリで文書化する必要があります。Ansible は、Azure など一部のシステムで動的インベントリに対応するため、ホスト インベントリはランタイムで動的に生成されます。

ラボは以下の高レベルの手順で構成されます。

- Azure Cloud Shell を使用して 2 台の Azure VM をデプロイする
- ダイナミック インベントリを生成する
- Azure CLI を使用して Azure で Ansible Control VM を作成する
- Ansible を Azure VM にインストールして構成する
- Ansible の構成とサンプル プレイブック ファイルをダウンロードする
- Azure AD でサービス プリンシパルを作成して構成する
- Ansible で使用できるように Azure AD 資格情報と SSH を構成する
- Ansible プレイブックを使用して Azure VM をデプロイする
- Ansible プレイブックを使用して Azure VM を構成する
- Ansible プレイブックを使用して Azure VM で構成と希望する状態の管理を行う
- Ansible と Azure Resource Manager テンプレートを使用して、Azure リソースの構成管理と希望する状態を促進する

## 目標

このラボを完了すると、次のことができるようになります。

- Azure Cloud Shell を使用して Azure VM をデプロイする
- Ansible 動的インベントリを生成する
- Azure CLI を使用して Azure で Ansible Control VM を作成する
- Ansible を Azure VM にインストールして構成する
- Ansible の構成とサンプル プレイブック ファイルをダウンロードする
- Azure AD でサービス プリンシパルを作成して構成する
- Ansible で使用できるように Azure AD 資格情報と SSH を構成する
- Ansible プレイブックを使用して Azure VM をデプロイする
- Ansible プレイブックを使用して Azure VM を構成する
- Ansible プレイブックを使用して Azure VM で構成と希望する状態の管理を行う
- Ansible と Azure Resource Manager テンプレートを使用して、Azure リソースの構成管理と希望する状態を促進する

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

### 演習 1: Ansible を使用して Azure VM をデプロイ、構成、管理する

この演習では、Ansible を使用して Azure リソースをデプロイ、構成、管理します。

#### タスク 1: Azure Cloud Shell を使用して 2 台の Azure VM を作成する

このタスクでは、Azure Cloud Shell を使用して 2 台の Azure VM を作成します。

1.  ラボのコンピューターから Web ブラウザーを起動し、[**Azure Portal**](https://portal.azure.com) に移動します。このラボで使用している Azure サブスクリプションで少なくとも共同作成者のロールがあるユーザー アカウントを使ってサインインします。
1.  Azure portal のツールバーで、検索テキスト ボックスのすぐ右側にある **Cloud Shell** アイコンをクリックします。 

    > **注**: [https://shell.azure.com](https://shell.azure.com) に移動して直接、Cloud Shell にアクセスすることもできます。

1.  **Bash** または **PowerShell** のいずれかを選択するためのプロンプトが表示されたら、**「Bash」** を選択します。 

    > **注**: **Cloud Shell** を初めて起動し、「**ストレージがマウントされていません**」というメッセージが表示された場合は、このラボで使用しているサブスクリプションを選択し、「**ストレージの作成**」を選択します。 

1.  Cloud Shell ペインでバッシュ セッションから以下を実行し、Azure VM をホストするリソース グループを作成します (`<Azure_region` プレースホルダーを、このラボでデプロイしたい Azure リージョンの名前に置き換えます):

    ```bash
    LOCATION=<Azure_region>
    RGNAME=az400m14l03arg
    az group create --name $RGNAME --location $LOCATION
    ```

1.  Cloud Shell ペインのバッシュ セッションから以下を実行し、Ubuntu を実行している最初の Azure VM を、前のステップで作成したリソース グループにデプロイします:

    ```bash
    VM1NAME=az400m1403vm1
    az vm create --resource-group $RGNAME --name $VM1NAME --image UbuntuLTS --generate-ssh-keys --no-wait
    ```

    > **注**: デプロイが完了するのを待たずに、次の手順に進みます。 

1.  Cloud Shell ペインのバッシュ セッションから以下を実行し、Ubuntu を実行している 2 番目の Azure VM を同じリソース グループにデプロイします:

    ```bash
    VM2NAME=az400m1403vm2
    az vm create --resource-group $RGNAME --name $VM2NAME --image UbuntuLTS --generate-ssh-keys
    ```

    > **注**: デプロイが完了するのを待ってから、次の手順に進みます。2 分間程度かかる場合があります。

1.  Cloud Shell ペインのバッシュ セッションから以下を実行し、最初の Azure VM に Nginx のタグを付け、Nginx Web サーバーとして識別します:

    ```bash
    VM1ID=$(az vm show --resource-group $RGNAME --name $VM1NAME --query id --output tsv)
    az resource tag --tags Ansible=nginx --id $VM1ID
    ```

#### タスク 2: 動的インベントリの生成

このタスクでは、前のタスクでデプロイした 2 台の Azure VM をターゲットにして動的 Ansible インベントリを生成します。

> **注**: Ansible 2.8 以降では、Azure 動的インベントリ プラグインが提供されています。

1.  Cloud Shell ペインのバッシュ セッションから以下を実行し、「**myazure_rm.yml**」という名前の新しいファイルを作成し、Nano テキスト エディターでこれを開きます。

    ```bash
    nano ./myazure_rm.yml
    ```

1.  Nano エディター インターフェイス内で以下の内容を貼り付けます:

    ```bash
    plugin: azure_rm
    include_vm_resource_groups:
    - az400m14l03arg
    auth_source: auto

    keyed_groups:
    - prefix: tag
      key: tags
    ```

1.  Nano エディター インターフェイス内で **ctrl + o** キーの組み合わせを押し、**Enter** キーを押してから **ctrl + x** キーの組み合わせを押して変更を保存し、ファイルを閉じます。
1.  Cloud Shell ペインのバッシュ セッションから以下を実行して Ansible Azure モジュールをインストールします:

    ```bash
    curl -O https://raw.githubusercontent.com/ansible-collections/azure/dev/requirements-azure.txt
    pip install -r requirements-azure.txt
    rm requirements-azure.txt
    ansible-galaxy collection install azure.azcollection
    ```

    > **注**: Ansible バージョン 2.10 以降では、Azure モジュールはコア モジュールとは別個に維持されます。Ansible のバージョンを確認するには、`ansible --version` を実行してください。

1.  Cloud Shell ペインのバッシュ セッションで、`exit` と入力し、**Enter** キーを押してセッションを終了します。 

    > **注**: これは、モジュールのインストールを有効にするために必要です。 

1.  Azure portal のツールバーで **Cloud Shell** アイコンをkる一句氏、Cloud Shell セッションを再開します。 
1.  Cloud Shell ペインのバッシュ セッションから以下を実行し、ping テストを行い、**myazure_rm.yml** ファイルに名前を含めたリソース グループで Azure で実行中のすべての仮想マシンの動的インベントリを生成します:

    ```bash
    ansible all -m ping -i ./myazure_rm.yml
    ```

    > **注**: コマンドを初めて実行する際は、ターゲット VM の信頼性を認識する必要があります。「**yes**」と入力してから **Enter** キーを押してください。これは各ターゲット Azure VM で行う必要があるかもしれません。信頼性の確認後、コマンドのその後の実行は正常に返され、確認する必要はありません。必要であれば上記のコマンドを数回実行し、成功する出力を生成します。

    > **注**: 非推奨警告は無視してください。 

1.  Cloud Shell ペインのバッシュ セッションから以下を実行し、インベントリでホストを一覧表示します:

    ```bash
    ansible all -i ./myazure_rm.yml --list-hosts
    ```

1.  Cloud Shell ペインのバッシュ セッションから以下を実行し、インベントリでタグに基づくホストのグループ化を一覧表示します:

    ```bash
    ansible-inventory -i ./myazure_rm.yml --graph
    ```

1.  Cloud Shell ペインのバッシュ セッションから以下を実行し、インベントリで特定のタグ値があるホストすべてへの接続性をテストします:

    ```bash
    ansible -i ./myazure_rm.yml -m ping tag_Ansible_nginx
    ```

1.  Cloud Shell ペインのバッシュ セッションから以下を実行し、最初のホストのプロパティの JSON ベースの表記を一覧表示します (`<Ansible_host_name>` は、`ansible all -i ./myazure_rm.yml --list-hosts` コマンドで返された最初のホストの名前に置き換えます):

    ```bash
    ansible-inventory -i ./myazure_rm.yml --host <Ansible_host_name> | jq
    ```

1.  Cloud Shell ペインのバッシュ セッションから以下を実行し、インベントリのあらゆるホストのプロパティの JSON ベースの表記を一連表示します:

    ```bash
    ansible-inventory -i ./myazure_rm.yml --list | jq
    ```

1.  「Cloud Shell」 ペインを閉じます。

#### タスク 3: Ansible コントロール マシンとして機能する Azure VM のプロビジョニングを行う

このタスクでは、Azure CLI を使用して Azure VM をデプロイし、Ansible 環境を管理できるように構成します。

> **注**: Ansible 静的ホスト インベントリは **etc/ansible/hosts** ファイルを使用します。Azure Cloud Shell は非動的インベントリで Ansible 管理を実装するためのルート アクセスを提供せず、ストレージ オプションがユーザーの **$Home** ディレクトリに制限されるため、Linux を実行している Azure VM をデプロイして、Ansible 管理システムとして構成します。

1.  Azure portal のツールバーで、検索テキスト ボックスのすぐ右側にある **Cloud Shell** アイコンをクリックします。 

    > **注**: [https://shell.azure.com](https://shell.azure.com) に移動して直接、Cloud Shell にアクセスすることもできます。

1.  Cloud Shell ペインでバッシュ セッションから以下を実行し、新しい Azure VM をホストするリソース グループを作成します:

    ```bash
    RG1NAME=az400m14l03arg
    VM1NAME=az400m1403vm1
    LOCATION=$(az vm show --resource-group $RG1NAME --name $VM1NAME --query location --output tsv)
    RG2NAME=az400m14l03brg
    az group create --name $RG2NAME --location $LOCATION
    ```

1.  Cloud Shell ペインでバッシュ セッションから以下を実行し、Ansible 管理システムとして機能する Azure VM をホストする仮想ネットワークを作成します:

    ```bash
    VNETNAME=az400m1403-vnet
    SUBNETNAME=ansible-subnet
    az network vnet create \
    --name $VNETNAME \
    --resource-group $RG2NAME \
    --location $LOCATION \
    --address-prefixes 192.168.0.0/16 \
    --subnet-name $SUBNETNAME \
    --subnet-prefix 192.168.1.0/24
    ```

1.  Cloud Shell ペインでバッシュ セッションから以下を実行し、Ansible 管理システムとして機能する Azure VM のネットワーク アダプターに割り当てるパブリック IP アドレスを作成します:

    ```bash
    PIPNAME=az400m1403-pip
    az network public-ip create \
    --name $PIPNAME \
    --resource-group $RG2NAME \
    --location $LOCATION
    ```
 
1.  Cloud Shell ペインでバッシュ セッションから以下を実行し、Ansible コントロール マシンとして機能する Azure VM を作成します:

    ```bash
    VM3NAME=az400m1403vm3
    az vm create \
    --name $VM3NAME \
    --resource-group $RG2NAME \
    --location $LOCATION \
    --image UbuntuLTS \
    --vnet-name $VNETNAME \
    --subnet $SUBNETNAME \
    --public-ip-address $PIPNAME \
    --authentication-type password \
    --admin-username azureuser \
    --admin-password Pa55w.rd1234
    ```

    > **注**: デプロイが完了するのを待ってから、次の手順に進みます。2 分間程度かかる場合があります。

    > **注**: プロビジョニングの完了後、JSON ベースの出力で、出力に含まれている **"publicIpAddress」** プロパティの値を識別します。 

1.  Cloud Shell ペインでバッシュ セッションから以下を実行し、SSH を使用して新しくデプロイされた Azure VM に接続します:

    ```bash
    PIP=$(az vm show --show-details --resource-group $RG2NAME --name $VM3NAME --query publicIps --output tsv)
    ssh azureuser@$PIP
    ```

1.  確認して続行するよう指示されたら、「**yes**」と入力して **Enter** キーを押します。パスワードを入力するよう指示されたら「**Pa55w.rd1234**」と入力します。

#### タスク 4: Ansible を Azure VM にインストールして構成する

このタスクでは、前のタスクでデプロイした Azure VM で Ansible をインストールして構成します。

1.  Cloud Shell ペインのバッシュ セッションで、新しくデプロイされた Azure VM へのSSH セッション内で以下を実行し、最新バージョンとパッケージの詳細が含まれるように Advanced Packaging Tool (apt) パッケージのリストを更新します:

    ```bash
    sudo apt update
    ```

1.  Cloud Shell ペインのバッシュ セッションで、新しくデプロイされた Azure VM へのSSH セッション内で以下を実行し、Ansible で必要な前提条件をインストールします:

    ```bash
    sudo apt-get install -y libssl-dev libffi-dev python-dev python-pip
    ```

1.  Cloud Shell ペインのバッシュ セッションで、新しくデプロイされた Azure VM へのSSH セッション内で以下を実行し、Ansible と必要な Azure モジュールをインストールします:

    ```bash
    sudo apt-get install python3-pip
    sudo apt install ansible
    curl -O https://raw.githubusercontent.com/ansible-collections/azure/dev/requirements-azure.txt
    pip3 install -r requirements-azure.txt
    rm requirements-azure.txt
    ansible-galaxy collection install azure.azcollection
    ```

1.  Cloud Shell ペインのバッシュ セッションで、新しくデプロイされた Azure VM へのSSH セッション内で以下を実行し、dnspython パッケージをインストールして Ansible プレイブックがデプロイ前に DNS 名を確認できるようにします:

    ```bash
    sudo pip3 install dnspython 
    ```

1.  Cloud Shell ペインのバッシュ セッションで、新しくデプロイされた Azure VM へのSSH セッション内で以下を実行し、**jq** JSON 解析ツールをインストールします:

    ```bash
    sudo apt install jq
    ```

1.  Cloud Shell ペインのバッシュ セッションで、新しくデプロイされた Azure VM へのSSH セッション内で以下を実行し、Azure CLI をインストールします:

    ```sh
    curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
    ```

#### タスク 5: Ansible の構成とサンプル プレイブック ファイルをダウンロードする

このタスクでは、サンプル ラボ ファイルとともに GitHub Ansible 構成リポジトリからダウンロードします。 

1.  Cloud Shell ペインのバッシュ セッションで、新しくデプロイされた Azure VM へのSSH セッション内で以下を実行し、**git** がインストールされていることを確認します:

    ```bash
    sudo apt install git
    ```

1.  Cloud Shell ペインのバッシュ セッションで、新しくデプロイされた Azure VM へのSSH セッション内で以下を実行し、 GitHub から ansible ソース コードを複製します:

    ```bash
    git clone git://github.com/ansible/ansible.git --recursive
    ```

1.  Cloud Shell ペインのバッシュ セッションで、新しくデプロイされた Azure VM へのSSH セッション内で以下を実行し、 GitHub から PartsUnlimitedMRP リポジトリを複製します:

    ```bash
    git clone https://github.com/Microsoft/PartsUnlimitedMRP.git
    ```

    > **注**: このリポジトリには、広範なリソースを作成するためのプレイブックが含まれており、その一部をこのラボで使用します。

#### タスク 6: Azure Active Directory のサービス プリンシパルを作成して構成する

このタスクでは、Azure AD サービス プリンシパルを生成して、Azure リソースへのアクセスに必要な Ansible の非対話型認証を促します。また、前のタスクで作成したリソース グループでサービス プリンシパルに共同作成者のロールを割り当てます。

1.  Cloud Shell ペインのバッシュ セッションで、新しくデプロイされた Azure VM への SSH セッション内で以下を実行し、お使いになっている Azure サブスクリプションに関連付けられている Azure AD テナントにサインインします:

    ```bash
    az login
    ```

1.  以前のコマンドの出力として表示されているコードを確認し、ラボのコンピューターに切り替えます。ラボのコンピューターから Azure portal が表示されているブラウザー ウィンドウで別のタブを開き、[Microsoft デバイス ログイン ページ](https://microsoft.com/devicelogin) に移動します。指示されたらコードを入力し、「**次へ**」 を選択します。

1.  指示されたら、このラボで使用している資格情報を使ってサインインし、ブラウザー タブを閉じます。

1.  Cloud Shell ペインのバッシュ セッションに戻り、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し、サービス プリンシパルを生成します:

    ```bash
    ANSIBLESP=$(az ad sp create-for-rbac --name az400m1403Ansible)
    ```

1.  Cloud Shell ペインのバッシュ セッションで、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し、新しく生成されたサービス プリンシパルのプロパティを表示します:

    ```bash
    echo $ANSIBLESP | jq
    ```

1.  出力をレビューし、**appId**、**password**、**tenant** プロパティの値を記録します。出力は次の形式にする必要があります:

    ```bash
    {
      "appId": "0df61458-1a6e-4b4c-b02a-d4ae43cd981e",
      "displayName": "az400m1403Ansible",
      "name": "http://az400m1403Ansible",
      "password": "JAHxIbRUypM~-DiA27Guj58E4nC.S_u~_U",
      "tenant": "44bb87a9-c9c8-4c0f-b176-88d7814533ba"
    }
    ```

1.  Cloud Shell ペインのバッシュ セッションで、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し、サブスクリプションの値を特定します:

    ```bash
    SUBSCRIPTIONID=$(az account show --query id --output tsv)
    echo $SUBSCRIPTIONID
    ```

1.  Cloud Shell ペインのバッシュ セッションで、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し、組み込み Azure ロール ベースのアクセス制御共同作成者のロールの **ID** プロパティの値を取得します: 

    ```bash
    CONTRIBUTORID=$(az role definition list --name "Contributor" --query "[].id" --output tsv)
    echo $CONTRIBUTORID
    ```

1.  Cloud Shell ペインのバッシュ セッションで、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し、このラボで以前に作成されたリソース グループで共同作成者のロールを割り当てます: 

    ```bash
    APPID=$(echo $ANSIBLESP | jq '.appId' -r)

    RG2NAME=az400m14l03brg
    az role assignment create --assignee "$APPID" \
    --role "$CONTRIBUTORID" \
    --resource-group "$RG2NAME"
    ```

#### タスク 7: Ansible で使用できるように Azure AD 資格情報と SSH を構成する

このタスクでは、Ansible および Ansible で使用する SSH 向けの Azure アクセスを構成します。前者では、前のタスクで作成した Azure AD サービス プリンシパルを利用します。

> **注**: 指定された Ansible コントロール システムからのリモート管理を可能にするには、資格情報ファイルを作成するか、サービス プリンシパルの詳細を Ansible 環境変数としてエクスポートできます。この 2 つのオプションのうち前者を選択します。資格情報は、**~/.azure/credentials** ファイルに保管されます。 

1.  Cloud Shell ペインのバッシュ セッションで、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し、資格情報ファイルを作成します: 

    ```bash
    echo "[default]" > ~/.azure/credentials
    ```

1.  Cloud Shell ペインのバッシュ セッションで、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し、以前のタスクで特定した Azure サブスクリプションの ID を含む新しいラインを Ansible 資格情報ファイルに追加します: 

    ```bash
    echo subscription_id=$SUBSCRIPTIONID >> ~/.azure/credentials
    ```

1.  Cloud Shell ペインのバッシュ セッションで、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し、以前のタスクで特定したサービス プリンシパルの ID を含む新しいラインを Ansible 資格情報ファイルに追加します: 

    ```bash
    echo client_id=$APPID >> ~/.azure/credentials
    ```
1.  Cloud Shell ペインのバッシュ セッションで、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し、以前のタスクで特定したサービス プリンシパル ID のシークレットを含む新しいラインを Ansible 資格情報ファイルに追加します: 

    ```bash
    SECRET=$(echo $ANSIBLESP | jq '.password' -r)
    echo secret=$SECRET >> ~/.azure/credentials
    ```

1.  Cloud Shell ペインのバッシュ セッションで、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し、以前のタスクで特定した Azure AD テナントの ID を含む新しいラインを Ansible 資格情報ファイルに追加します: 

    ```bash
    TENANT=$(echo $ANSIBLESP | jq '.tenant' -r)
    echo tenant=$TENANT >> ~/.azure/credentials
    ```

1.  Cloud Shell ペインのバッシュ セッションで、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し、Ansible 資格情報ファイルを内容を確認します: 

    ```bash
    cat  ~/.azure/credentials
    ```

    > **注**: 出力は次のようになります。

    ```bash
    [default]
    subscription_id=1cfb08bc-a39e-46cc-9b1e-38a182b43d60
    client_id=0df61458-1a6e-4b4c-b02a-d4ae43cd981e
    secret=JAHxIbRUypM~-DiA27Guj58E4nC.S_u~_U
    tenant=44bb87a9-c9c8-4c0f-b176-88d7814533ba
    ```

    > **注**: ここでリモート SSH 接続用のパブリック / プライベート キーの組を作成し、その操作をテストします。 

1.  Cloud Shell ペインのバッシュ セッションで、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し、キーの組を生成します: 指示されたら、**Enter** キーを押し、ファイルの場所の既定値を受け入れます。パスフレーズは設定しません。

    ```bash
    ssh-keygen -t rsa
    ```

1.  Cloud Shell ペインのバッシュ セッションで、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し、プライベート キーをホストしている **ssh** フォルダーで読み取り、書き込み、実行許可を付与します:

    ```bash
    chmod 755 ~/.ssh
    ```

1.  Cloud Shell ペインのバッシュ セッションで、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し、**authorized_keys** ファイルで読み取りおよび書き込み許可を作成して設定します:

    ```bash
    touch ~/.ssh/authorized_keys
    chmod 644 ~/.ssh/authorized_keys
    ```

    > **注**: このファイルに含まれているキーを提供すると、パスワードがなくてもアクセスが許可されます。

1.  Cloud Shell ペインのバッシュ セッションで、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し、**authorized_keys** ファイルにパスワードを追加します:

    ```bash
    ssh-copy-id azureuser@127.0.0.1
    ```

1. 指示されたら「**yes**」と入力し、このラボで以前に 3 番目の Azure VM をデプロイした際に指定した **azureuser** ユーザー アカウントのパスワード「**Pa55w.rd1234**」を入力します。 
1.  Cloud Shell ペインのバッシュ セッションで、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し、パスワードの入力を指示されていないことを確認します:

    ```bash
    ssh 127.0.0.1
    ```
1.  Cloud Shell ペインのバッシュ セッションで、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で「**exit**」と入力して **Enter** キーを押し、設定したばかりのループバック接続を終了します: 

> **注**: パスワードのない SSH 認証を設定することは、Ansible 環境を設定するための重要なステップです。 

#### タスク 8: Ansible プレイブックを使用して Web サーバー Azure VM を作成する

このタスクでは、Ansible プレイブックを使用して Web サーバーをホストする Azure VM を作成します。 

> **注**: Ansible がコントロール Azure VM で実行中になったので、最初のプレイブックをデプロイして管理対象 Azure VM を作成して構成することができます。プレイブックは、動的インベントリではなく、localhost ファイルを使用します。デプロイでは、サンプル プレイブック「**~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_vm_web.yml**」を使用します。これは、このラボで以前に Github リポジトリから複製されていたものです。サンプル プレイブックをデプロイするっ前に、コンテンツに含まれているパブリック SSH キーを、以前のタスクで生成したキーに置き換える必要があります。 

1.  Cloud Shell ペインのバッシュ セッションで、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し、以前のタスクで生成され、ローカルに保管されているパブリック キーを識別します:

    ```bash
    cat ~/.ssh/id_rsa.pub
    ```

1.  出力を記録します (出力文字列の最後のユーザー名を含む)。 
1.  Cloud Shell ペインのバッシュ セッションで、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し、Nano テキスト エディターで **new_vm_web.yml** ファイルを開きます:

    ```bash
    nano ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_vm_web.yml
    ```

1.  Nano エディター インターフェイス内で、ファイルの最後の方にある SSH 文字列を見つけ (key_data 値)、このファイルに含まれているキーを削除します。
1.  Nanoエディターで、ファイルの最後の方にある SSH 文字列を見つけ (`key_data` エントリ)、既存のキー値を削除して、このタスクで以前に記録しておいたキー値に置き換えます。 
1.  ファイルに含まれている `admin_username` エントリの値が、Ansible コントロール システムをホストしている Azure VM へのサインインで使われたユーザー名に一致することを確認します (**azureuser**)。`Ssh_public_keys` セクションの `path` エントリで同じユーザー名を使用する必要があります。
1.  `vm_size` エントリの値を `Standard_A0` から `Standard_D2s_v3` に変更します。
1.  必要に応じて、`dnsname: '{{ vmname }}.westeurope.cloudapp.azure.com'` エントリでリージョンの名前を、デプロイのターゲットである Azure リージョンの名前に変更します。
1.  Nano エディター インターフェイス内で **ctrl + o** キーの組み合わせを押し、**Enter** キーを押してから **ctrl + x** キーの組み合わせを押して変更を保存し、ファイルを閉じます。

    > **注**: ラボの最初に作成したリソース グループに Azure VM をデプロイします。デプロイでは以下の値を使用してください: 

    | 設定 | 値 |
    | --- | --- |
    | リソース グループ | **az400m14l03arg** |
    | バーチャル ネットワーク | **az400m1403vm1VNET** |
    | サブネット | **az400m1403vm1Subnet** |

    > **注**: 注: 変数は、プレイブック内で定義するか、ランタイムに `ansible-playbook` コマンドを呼び出した際、`--extra-vars` オプションを含めると入力できます。VM 名には、最高 15 字の小文字と数字のみを使用できます (ハイフン、下線、大文字は使用不可)。同じ名前を使用して、該当する Azure VM に関連のあるパブリック IP アドレスの DNS 名を生成するため、グローバルに一意であることを確認してください。 

1.  Cloud Shell ペインのバッシュ セッションで、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し、Azure VM をプロビジョニングしているサンプル ansible プレイブックをデプロイします (`<VM_name>` プレースホルダーは、選択した一意の VM 名を示します):

    ```bash
    ansible-playbook ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_vm_web.yml --extra-vars "vmname=<VM_name> resgrp=az400m14l03arg vnet=az400m1403vm1VNET subnet=az400m1403vm1Subnet"
    ```

    > **注**: 無効な VM 名を入力すると、以下のエラーが返される可能性があります:

    - `fatal: [localhost]: FAILED! => {"changed": false, "failed": true, "msg": "The storage account named storageaccountname is already taken. - Reason.already_exists"}`. これを解決するには、Azure VM の別の名前を使ってください。使用された名前はグローバルに一意ではありません。
    - `fatal: [localhost]: FAILED! => {"changed": false, "failed": true, "msg": "Error creating or updating your-vm-name - Azure Error: InvalidDomainNameLabel\nMessage: VM のドメイン名ラベルは無効です。以下の正規表現に従う必要があります: ^「a-z」「a-z0-9-」{1,61}「a-z0-9」$.”}`. この問題を解決するには、必要な命名規則に従って、Azure VM に別の名前を使用してください。 

    > **注**: デプロイが完了するのを待ちます。3 分間程度かかる場合があります。 

    > **注**: デプロイの完了後、動的インベントリを実行し、Ansible が新しい VM を検出することを確認できます。 

1.  Cloud Shell ペインのバッシュ セッションで、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し、動的インベントリを生成します:

    ```bash
    python /usr/local/lib/python2.7/dist-packages/ansible_collections/community/general/scripts/inventory/azure_rm.py --list | jq
    ```
1.  Cloud Shell ペインのバッシュ セッションで、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し、ping テストを行って、動的インベントリ ファイルに新しくデプロイされた Azure VM が含まれていることを確認します: 以前と同様、このテストを初めて実行する際は SSH ホスト キーを確認する必要がありますが、その後の実行では操作は不要です。

    ```bash
    sudo chmod +x /usr/local/lib/python2.7/dist-packages/ansible_collections/community/general/scripts/inventory/azure_rm.py
    ansible -i /usr/local/lib/python2.7/dist-packages/ansible_collections/community/general/scripts/inventory/azure_rm.py all -m ping
    ```

    > **注**: Azure Cloud Shell を介して以前にデプロイされた 2 台の Azure VM はインベントリに表示されますが、Ansible コントロール マシンで生成されたパブリック キーが含まれていないため、アクセスすることはできません。新しくデプロイされた Azure VM から応答を受け取った後、次のタスクに進みます。 

1.  動的インベントリをうまく完了できない場合は、Cloud Shell ペインのバッシュ セッションで、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し、SSH を使用することで、新しくデプロイされた Azure VM に接続します (`<VM_name>` プレースホルダーは、新しくプロビジョニングされた Azure VM に割り当てた名前を示します):

    ```bash
    $RG2NAME='az400m14l03arg'
    $VM4NAME='<VM_name>'
    PIP=$(az vm show --show-details --resource-group $RG2NAME --name $VM4NAME --query publicIps --output tsv)
    ssh azureuser@$PIP
    ```

1.  続行を確認するよう指示されたら、「**yes**」と入力して **Enter** キーを押します。接続が確立されたら、「**exit**」と入力して再び **Enter** キーを押し、終了します。

#### タスク 9: Ansible プレイブックを使用して、新しくデプロイされた Azure VM を構成する

このタスクでは、別の Ansible プレイブックを実行し、新しく作成されたマシンを構成します。ソフトウェア パッケージ httpd をインストールし、HTML ページを Github リポジトリからダウンロードするプレイブックを使用します。これが完了すると、Web サーバーは完全に機能するようになります。

> **注**: サンプル プレイブック「**~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/httpd.yml**」を使用します。プレイブックのホスト パラメーターを変更するために **vmname** 変数を利用します。これは、プレイブックが (動的インベントリのスクリプトから返されたホストのうち) どのホストをターゲットにするのかを定義するものです。 

1.  Cloud Shell ペインのバッシュ セッションで、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し実行し、新しくデプロイされた Azure VM が現在、どの Web サービスも実行していないことを確認します (`<IP_address>` プレースホルダーは、新しくプロビジョニングされた Azure VM のネットワーク アダプターに割り当てられたパブリック IP アドレスを示します):

    ```bash
    curl http://<IP_address>
    ```

    > **注**: このパブリック IP アドレスは、以下のスクリプトを実行すると識別できます (`<VM_name>` プレースホルダーは、新しくプロビジョニングされた Azure VM に割り当てた名前を示します):

    ```bash
    $RG2NAME='az400m14l03brg'
    $VM4NAME='<VM_name>'
    PIP=$(az vm show --show-details --resource-group $RG2NAME --name $VM4NAME --query publicIps --output tsv)
    echo $PIP
    ```

1.  応答が `curl: (7) Failed to connect to 52.186.157.26 port 80: Connection refused` 形式であることを確認します。
1.  Cloud Shell ペインのバッシュ セッションから、Ansible コントロール マシンとして構成されている Azure VM へのSSH セッション内で以下を実行し、Ansible プレイブックを使用して HTTP サービスをインストールします (`<VM_name>` プレースホルダーは、新しくプロビジョニングされた Azure VM に割り当てた名前を示します): 

    ```bash
    ansible-playbook -i /usr/local/lib/python2.7/dist-packages/ansible_collections/community/general/scripts/inventory/azure_rm.py  ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/httpd.yml --extra-vars "vmname=<VM_name>"
    ```

    > **注**: インストールが完了するまで待ちます。これには 1 分もかかりません。 

1.  インストールの完了後、Cloud Shell ペインのバッシュ セッションで、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し、新しくデプロイされた Azure VM が現在、Web サービスを実行していることを確認します (`<IP_address>` プレースホルダーは、新しくプロビジョニングされた Azure VM のネットワーク アダプターに割り当てられたパブリック IP アドレスを示します):

    ```bash
    curl http://<IP_address>
    ```

    > **注**: 出力には以下の内容が必要です: 

    ```html
     <!DOCTYPE html>
     <html lang="en">
         <head>
             <meta charset="utf-8">
             <title>Hello World</title>
         </head>
         <body>
             <h1>Hello World</h1>
             <p>
                 <br>This is a test page
                 <br>This is a test page
                 <br>This is a test page
             </p>
         </body>
     </html>
    ```

#### タスク 10: Ansible プレイブックを使用して、Azure で構成管理と希望する状態を実装する

このタスクでは、Ansible プレイブックを使用して、Azure で構成管理と希望する状態を実装します。

> **注**: 定期的に `ansible-playbook` コマンドを実行して、構成が該当するプレイブックの内容に一致していることを確認できます。これを自動的に行うには、Linux cron 機能を利用します。このタスクでは、毎分、コマンドを実行しますが、運用環境ではおそらく頻度を下げることになります。

> **注**: Ansible プレイブックを使用して cron ジョブを設定します。 

1.  Cloud Shell ペインのバッシュ セッションから、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し、Nano テキスト エディターで Ansible 構成ファイルを開きます:

    ```bash
    sudo nano /etc/ansible/ansible.cfg
    ```

1.  Nano エディター インターフェイス内の `「Default」` セクションで、最初のハッシュ文字 `#` をライン `#host_key_checking = False` から削除します
1.  Nano エディター インターフェイス内で **ctrl + o** キーの組み合わせを押し、**Enter** キーを押してから **ctrl + x** キーの組み合わせを押して変更を保存し、ファイルを閉じます。
1.  Cloud Shell ペインのバッシュ セッションから、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し、Nano テキスト エディターで cron ジョブを構成するプレイブックを開きます:

    ```bash
    nano ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/cron.yml
    ```

1.  Nano エディター インターフェイス内の `job` エントリで、`< your-vm-name >` プレースホルダーを、前のタスクで構成した Azure VM の名前に置き換えます。 
1.  Nano エディター インターフェイス内で、`job` エントリを `ansible-playbook -i /usr/local/lib/python2.7/dist-packages/ansible_collections/community/general/scripts/inventory/azure_rm.py  /home/azureuser/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/httpd.yml --extra-vars "vmname=<VM_name>"` に設定します。`<VM_name>` プレースホルダーは、管理対象 Azure VM に割り当てた名前を示します。
1.  Nano エディター インターフェイス内で **ctrl + o** キーの組み合わせを押し、**Enter** キーを押してから **ctrl + x** キーの組み合わせを押して変更を保存し、ファイルを閉じます。
1.  Cloud Shell ペインのバッシュ セッションから、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し、cron ジョブを作成します:

    ```bash
    ansible-playbook ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/cron.yml
    ```

1.  Cloud Shell ペインのバッシュ セッションから、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し、cron ジョブが実行中であることを確認します:

    ```bash
    sudo tail /var/log/syslog
    ```

    > **注**: ユーザー全員の cron ジョブはすべて同じファイルにログされるため、root 権限が必要になります。 

1.  Cloud Shell ペインのバッシュ セッションから、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し、cron エディター インターフェイスから cron ジョブを識別します:

    ```bash
    crontab -e
    ```

1.  指示されたら、nano 向けに **1** を選択し、**Enter** キーを押します。
1.  **Crontab** エディター内で、**usr/bin/ansible-playbook** エントリがあることを確認してから **ctrl + x** キーの組み合わせを押してエディターを閉じます。

    > **注**: 5 個のアスタリスクは、ジョブが毎分実行されることを示します。

    > **注**: セットアップが機能するか確認してみましょう。 

1.  Cloud Shell ペインのバッシュ セッションから、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内で以下を実行し実行し、新しくデプロイされた Azure VM が現在、どの Web サービスも実行していないことを確認します (`<Ip_address>` プレースホルダーは、新しくプロビジョニングされた Azure VM のネットワーク アダプターに割り当てられたパブリック IP アドレスを示します):

    ```bash
    ssh azureuser@<IP_address>
    ```

1.  Cloud Shell ペインのバッシュ セッションから、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内、および Web サーバーとして構成されている Azure VM への SSH セッション内で以下を実行し、Web サイトがまだ機能していることを確認します:

    ```bash
    curl http://127.0.0.1
    ```

1.  Cloud Shell ペインのバッシュ セッションから、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内、および Web サーバーとして構成されている Azure VM への SSH セッション内で以下を実行し、Web サイトのホーム ページを削除します:

    ```bash
    rm /var/www/html/index.html
    ```

1.  少し待った後、Cloud Shell ペインのバッシュ セッションから、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内、および Web サーバーとして構成されている Azure VM への SSH セッション内で以下を実行し、Web サイトのホーム ページが復元されたことを確認します:

    ```bash
    curl http://127.0.0.1
    ```

1.  Cloud Shell ペインのバッシュ セッションから、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内、および Web サーバーとして構成されている Azure VM への SSH セッション内で、「**exit**」と入力して **Enter** キーを押し、Ansible コントロール マシンへの SSH セッションに戻ります:

    > **注**: 定期的に ansible プレイブックを実行すると、Azure VM の状態を希望する構成から変えてしまう変更点を修正できます (プレイブックで定義)。このアプローチを使うことにｙろい、構成の逸脱を防ぎ、希望する状態を維持できます。また、同じアプローチで Ansible プレイブックを定期的に実行し、Azure リソースをターゲットにすることも可能です。これにより、インフラストラクチャがデプロイされ、意図されているとおりに構成されていることを確認できます。 

#### タスク 11: Ansible と Azure Resource Manager テンプレートを使用して、Azure リソースの構成管理と希望する状態を促進する

このタスクでは、Ansible と Azure Resource Manager テンプレートを使用して、Azure リソースの構成管理と希望する状態を促進します。

> **注**: 前のタスクで説明されているように、Ansible を使うと、該当するモジュールでサポートされている既存のリソースの構成逸脱を修正できます。ただし、Ansible を使用して、Azure Resource Manager テンプレートを参照するプレイブックをデプロイすることも可能です。これにより、Azure Resource Manager の提供する機能とリソースに直接アクセスできます。 

> **注**: 別の Azure VM をデプロイしますが、今回は Azure Resource Manager テンプレートを参照する ansible プレイブックを使用します。作業をシンプルにするため、[単一のストレージ アカウントをプロビジョニングする非常にわかりやすい Azure クイックスタート テンプレート](https://github.com/Azure/azure-quickstart-templates/tree/master/101-storage-account-create) を利用します。関連のあるプレイブックの構文は、**PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_ARM_deployment.yml** プレイブックで確認できます。

1.  Cloud Shell ペインのバッシュ セッションから、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内、および Web サーバーとして構成されている Azure VM への SSH セッション内で以下を実行し、Azure Resource Manager を呼び出すプレイブックを Nano テキスト エディターで開きます:

    ```bash
    nano  ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_ARM_deployment.yml
    ```

1.  Nano エディター インターフェイス内の `templateLink:` エントリで、現在の URL を `https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json` に置き換えます。
1.  Nano エディター インターフェイス内で **ctrl + o** キーの組み合わせを押し、**Enter** キーを押してから **ctrl + x** キーの組み合わせを押して変更を保存し、ファイルを閉じます。
1.  Cloud Shell ペインのバッシュ セッションから、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内、および Web サーバーとして構成されている Azure VM への SSH セッション内で以下を実行し、プレイブックを実行します (`<Azure_region>` プレースホルダーは、このラボであらゆるリソースをデプロイした Azure リージョンの名前に置き換えます):

    ```bash
    ansible-playbook ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_ARM_deployment.yml --extra-vars "resgrp=az400m14l03brg location=<Azure_region>"
    ```

    > **注**: ARM テンプレートのデプロイは冪等であると認識することが重要です。ARM テンプレートを一度デプロイすると、そのテンプレートを複数回デプロイした場合と同じ効果があります。つまり、ARM テンプレートを同じリソース グループに再び安全にデプロイでき、重複するリソースが作成されることはありません。実際に ARM テンプレートの再デプロイは定期的な間隔でスケジュールできます。たとえば、前のタスクで説明されているように、Linux cron ユーティリティを使用できます。 

1.  Cloud Shell ペインのバッシュ セッションから、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内、および Web サーバーとして構成されている Azure VM への SSH セッション内で以下を実行し、同じプレイブックを実行します (`<Azure_region>` プレースホルダーは、このラボであらゆるリソースをデプロイした Azure リージョンの名前に置き換えます)。

    ```bash
    ansible-playbook ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_ARM_deployment.yml --extra-vars "resgrp=az400m14l03brg location=<Azure_region>"
    ```

    > **注**: ここでテンプレートによってデプロイされたストレージ アカウントを修正します。このような変更は検出が難しい可能性がありますが、環境に悪影響を与えかねません。このため、自動的にこのような変更を希望する状態に戻す仕組みを整えておくと役に立ちます。この場合は、ストレージ アカウントのレプリケーション設定を変更します。 

1.  Cloud Shell ペインのバッシュ セッションから、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内、および Web サーバーとして構成されている Azure VM への SSH セッション内で以下を実行し、このタスクで以前にデプロイしたストレージ アカウントの現在の設定を識別します:

    ```bash
    RGNAME=az400m14l03brg
    STORAGEACCOUNTNAME=$(az storage account list --resource-group $RGNAME --query "[].name" --output tsv)
    az storage account show \
    --name $STORAGEACCOUNTNAME \
    --resource-group $RGNAME \
    --query sku
    ```

1.  Cloud Shell ペインのバッシュ セッションから、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内、および Web サーバーとして構成されている Azure VM への SSH セッション内で以下を実行し、SKU を `Standard_LRS` から `Standard_GRS` に変更して、変更が行われたことを確認します: 

    ```bash
    az storage account update \
    --name $STORAGEACCOUNTNAME \
    --resource-group $RGNAME \
    --sku 'Standard_GRS'

    az storage account show \
    --name $STORAGEACCOUNTNAME \
    --resource-group $RGNAME \
    --query sku
    ```

    > **注**: ここで、同じテンプレートをデプロイするプレイブックを再実行します。 

1.  Cloud Shell ペインのバッシュ セッションから、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内、および Web サーバーとして構成されている Azure VM への SSH セッション内で以下を実行し、同じプレイブックを再度実行します (`<Azure_region>` プレースホルダーは、このラボであらゆるリソースをデプロイした Azure リージョンの名前に置き換えます):

    ```bash
    ansible-playbook ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_ARM_deployment.yml --extra-vars "resgrp=az400m14l03brg location=<Azure_region>"
    ```

1.  Cloud Shell ペインのバッシュ セッションから、Ansible コントロール マシンとして構成されている Azure VM への SSH セッション内、および Web サーバーとして構成されている Azure VM への SSH セッション内で以下を実行し、変更が元に戻されたことを確認します: 

    ```bash
    az storage account show \
    --name $STORAGEACCOUNTNAME \
    --resource-group $RGNAME \
    --query sku
    ```

### 演習 3: Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングした Azure リソースを削除し、予期しない料金を排除します。 

> **注**: 新しく作成した Azure リソースのうち、使用しないリソースは必ず削除してください。使用しないリソースを削除しないと、予期しないコストが発生する場合があります。

#### タスク 1: Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。 

1.  Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```sh
    az group list --query "[?starts_with(name,'az400m14l03')].name" --output tsv
    ```

1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを削除します。

    ```sh
    az group list --query "[?starts_with(name,'az400m14l03')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    > **注**: コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

## レビュー

このラボでは、Ansible を使用して Azure リソースをデプロイ、構成、管理する方法を学習しました。 

