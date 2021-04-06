---
lab:
  title: "ラボ: Configuring Agent Pools and Understanding Pipeline Styles"
  module: "モジュール 5: Configuring Azureの構成"
---

# ラボ: エージェント プールの設定とパイプライン スタイルの理解

# 学生用ラボ マニュアル

## ラボの概要

YAML ベースのパイプラインを使用すると、CI/CD をコードとして完全に実装できます。パイプライン定義は、Azure DevOps プロジェクトの一部であるコードと同じリポジトリにあります。YAML ベースのパイプラインは、プルリクエスト、コードレビュー、履歴、分岐、テンプレートなど、従来のパイプラインの一部である幅広い機能をサポートします。

パイプライン スタイルの選択に関係なく、Azure Pipelines を使用してコードをビルドしたりソリューションをデプロイしたりするには、エージェントが必要です。エージェントは、一度に 1 つのジョブを実行するコンピューティング リソースをホストします。ジョブは、エージェントのホスト マシンで直接実行することも、コンテナーで実行することもできます。管理されている Microsoft がホストするエージェントを使用してジョブを実行するか、自分で設定および管理するセルフホステッド エージェントを実装するかを選択できます。

このラボでは、従来のパイプラインを YAML ベースのパイプラインに変換し、最初に Microsoft がホストするエージェントを使用して実行し、次にセルフホステッド エージェントを使用して同等のタスクを実行するプロセスを順を追って説明します。

## 目標

このラボを完了すると、次のことができるようになります。

- YAML ベースのパイプラインを実装する
- セルフホステッド エージェントを実装する

## ラボの所要時間

- 推定時間: **90 分**

## 手順

### 開始する前に

#### ラボの仮想マシンにログインする

次の資格情報を使用して、Windows 10 コンピューターにサインインしていることを確認します。

- ユーザー名: **Student**
- パスワード: **Pa55w.rd**

#### インストールされているアプリケーションを確認します

Windows 10 デスクトップでタスク バーを探します。タスク バーには、この課題で使用するアプリケーションのアイコンが含まれています:

- Microsoft Edge
- [Visual Studio Code](https://code.visualstudio.com/)

#### Azure DevOps 組織を設定する

このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/ja-jp/azure/devops/organizations/accounts/create-organization?view=azure-devops)の手順に従って作成してください。

### 演習 0: ラボの前提条件を構成する

この演習では、ラボの前提条件を設定します。Azure DevOps Demo Generator テンプレートに基づいて事前構成された Parts Unlimited チームプロジェクトで構成されます。

#### タスク 1: チーム プロジェクトを構成する

このタスクでは、Azure DevOps Demo Generator を使用して、**PartsUnlimited** テンプレートに基づいて新しいプロジェクトを生成します。

1.  ラボ コンピューターで、Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。このユーティリティ サイトは、ラボに必要なコンテンツ (作業項目、リポジトリなど) が事前に入力されたアカウント内に新しい Azure DevOps プロジェクトを作成するプロセスを自動化します。

    > **注**: サイトの詳細については、https://docs.microsoft.com/ja-jp/azure/devops/demo-gen を参照してください。

1.  「**サインイン**」 をクリックし、Azure DevOps サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
1.  必要な場合は、**Azure DevOps Demo Generator** ページで、「**承諾する**」 をクリックして、Azure DevOps サブスクリプションにアクセスするためのアクセス許可の要求を受け入れます。
1.  「**Create New Project**」 ページの 「**New Project Name**」 テキスト ボックスに「**Configuring Agent Pools and Understanding Pipeline Styles**」と入力し、「**Select organization**」 ドロップダウン リストで、Azure DevOps 組織を選択して、「**Choose a template**」 をクリックします。
1.  「**Choose a template**」 ページで、**PartsUnlimited** テンプレートをクリックし、「**Choose a template**」 をクリックします。
1.  「**Create Project**」 をクリックします。

    > **注**: プロセスが完了するまでお待ちください。これにはおよそ 2 分かかります。プロセスが失敗した場合は、DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

1.  「**Create New Project**」 ページで、「**Navigate to project**」 をクリックします

1.  [**Create a new project**]ページで、[**Navigate to project**]をクリックします。

### 演習 1：YAML ベースの AzureDevOps パイプラインを作成する

この演習では、従来の AzureDevOps パイプラインを YAML ベースのパイプラインに変換します。

#### タスク 1： Azure DevOps YAML パイプラインを作成する

このタスクでは、テンプレートベースの Azure DevOps YAML パイプラインを作成します。

1. **Configuring Agent Pools and Understanding Pipeline Styles**プロジェクトが開いた状態で AzureDevOps ポータルを表示している Web ブラウザーから、左側の垂直ナビゲーションペインで[**Pipelines**]をクリックします。
1. [**Pipelines**]ペインの[**Recent**]タブで、[**New pipeline**]をクリックします。
1. [**Where is your code?**]ペインで、**Azure Repos Git**をクリックします。
1. [**Select a repository**]ペインで、[**PartsUnlimited**]をクリックします。
1. [**Configure your pipeline**]ペインで、[**Starter pipeline**]をクリックします。
1. [**Review your pipeline YAML**]ペインで、サンプルパイプラインを確認し、[**Save and run**]ボタンの横にある下向きのキャレット記号をクリックし、[**Save**]をクリックします。
1. [**Save**]ペインで、[**Save**]をクリックします。

   > **注**：これにより、プロジェクトコードをホストするリポジトリのルートディレクトリに**azure-pipelines.yml**ファイルが作成されます。

   > **注**：サンプルの YAML パイプラインのコンテンツを、クラシックエディターによって生成されたパイプラインのコードに置き換え、クラシックパイプラインと YAML パイプラインの違いを考慮して変更します。

#### タスク 2：クラシックパイプラインを YAML パイプラインに変換する

このタスクでは、クラシックパイプラインを YAML パイプラインに変換します

1. 左側の垂直ナビゲーションペインで[**Pipelines**]をクリックします。
1. **Pipelines**ペインの**Recent**タブで、**PartsUnlimitedE2E**エントリの右端にマウスポインタを合わせて省略記号をクリックし、ドロップダウンメニューで[**Edit**]をクリックします。これにより、ラボの開始時に生成したプロジェクトの一部であるビルドパイプラインが表示されます。
1. **PartsUnlimitedE2E**編集ペインの[**Triggers**]をクリックし、[**PartsUnlimited**]ペインの右側にある[**Enable continuous integration**]チェックボックスをオフにします。 [**Save & queue**]ボタンの横にある下向きのキャレットをクリックし、ドロップダウンメニューで[**Save**]をクリックし、[**Save build pipeline**]で[**Save**]をクリックします。

   > **注**：これにより、このラボ中にリポジトリが変更されたために、意図しない自動ビルドが実行されるのを防ぐことができます。

   > **注**：または、コンテンツを新しいパイプラインにコピーしたら、既存のパイプラインを削除することもできます。

1. Azure DevOps ポータルの左側の垂直ナビゲーションペインの[**Pipelines**]をクリックします。
1. **Pipelines**ペインの**Recent**タブで、**PartsUnlimitedE2E**エントリをクリックします。
1. **PartsUnlimitedE2E**ペインの**Run pipeline**の横にある、縦の省略記号(3 つの縦のドット)記号をクリックし、ドロップダウンメニューで[**Export to YAML**]をクリックします。 これにより、**PartsUnlimitedE2E.yml**ファイルがローカルの**Downloads**フォルダーに自動的にダウンロードされます。

   > **注**：**Export to YAML** 機能は、AzureDevOps ポータル内のパイプラインエディターペインから利用できる古い**View YAML**オプションを置き換えます。これは、一度に 1 つのジョブの YAML コンテンツの表示に制限されていました。新しい機能は、YAML 解析ライブラリを含む既存のクラシックおよび YAML パイプラインインフラストラクチャを活用します。これにより、2 つの間のより正確な変換が実現します。次のパイプラインコンポーネントをサポートします。

   - 単一および複数のジョブ
   - チェックアウトオプション
   - 実行プランの並列処理
   - タイムアウトと名前のメタデータ
   - Demands
   - スケジュールおよびその他のトリガー
   - デフォルトとは異なるジョブを含むプールの選択
   - デフォルト入力の最適化を含む、すべてのタスクとすべての入力
   - ジョブとステップの条件
   - タスクグループの展開

   > **注**：新しい機能でカバーされていないコンポーネントは、変数とタイムゾーン変換のみです。YAML で定義された変数は、AzureDevOps ポータルのユーザーインターフェイスで設定された変数をオーバーライドします。**Export to YAML**機能がクラシックパイプライン変数の存在を検出した場合、それらは新しく生成された YAML パイプライン定義内のコメントに明示的に含まれます。同様に、YAML の cron スケジュールは UTC で表されるため、従来のスケジュールは組織のタイムゾーンに依存するため、それらの存在もコメントに含まれます。

   > **注**：この機能の詳細については、["View YAML"の置き換え](https://devblogs.microsoft.com/devops/replacing-view-yaml/)を参照してください。

1. ラボコンピューターで、Visual Studio Code を起動し、それを使用してファイル**PartsUnlimitedE2E.yml**を開きます。ファイルには次の内容が含まれている必要があります。

   ```yaml
   name: $(date:yyyyMMdd)$(rev:.r)
   jobs:
     - job: Phase_1
       displayName: Phase 1
       cancelTimeoutInMinutes: 1
       pool:
         vmImage: vs2017-win2016
       steps:
         - checkout: self
         - task: NuGetInstaller@0
           name: NuGetInstaller_1
           displayName: NuGet restore
           inputs:
             solution: '**\*.sln'
         - task: VSBuild@1
           name: VSBuild_2
           displayName: Build solution
           inputs:
             vsVersion: 15.0
             msbuildArgs: /p:DeployOnBuild=true /p:WebPublishMethod=Package /p:PackageAsSingleFile=true /p:SkipInvalidConfigurations=true /p:PackageLocation="$(build.stagingDirectory)" /p:IncludeServerNameInBuildInfo=True /p:GenerateBuildInfoConfigFile=true /p:BuildSymbolStorePath="$(SymbolPath)" /p:ReferencePath="C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\Common7\IDE\Extensions\Microsoft\Pex"
             platform: $(BuildPlatform)
             configuration: $(BuildConfiguration)
         - task: VSTest@1
           name: VSTest_3
           displayName: Test Assemblies
           inputs:
             testAssembly: '**\$(BuildConfiguration)\*test*.dll;-:**\obj\**'
             codeCoverageEnabled: true
             vsTestVersion: latest
             platform: $(BuildPlatform)
             configuration: $(BuildConfiguration)
         - task: CopyFiles@2
           name: CopyFiles1
           displayName: Copy Files
           inputs:
             SourceFolder: $(build.sourcesdirectory)
             Contents: "**/*.json"
             TargetFolder: $(build.artifactstagingdirectory)
         - task: PublishBuildArtifacts@1
           name: PublishBuildArtifacts_5
           displayName: Publish Artifact
           inputs:
             PathtoPublish: $(build.artifactstagingdirectory)
             TargetPath: '\\my\share\$(Build.DefinitionName)\$(Build.BuildNumber)'
   ```

1. Visual Studio Code ウィンドウのトップレベルメニューで[**Selection(選択)**]をクリックし、ドロップダウンメニューで[**Select All(すべて選択)**]をクリックします。
1. Visual Studio Code ウィンドウのトップレベルメニューで[**Edit**]をクリックし、ドロップダウンメニューで[**Copy(コピー)**]をクリックします。
1. Azure DevOps ポータルを表示しているブラウザーウィンドウに切り替え、左側の垂直ナビゲーションペインの[**Pipelines**]をクリックします。

1. **Pipelines**ペインの**Recent**タブで**PartsUnlimited**を選択し、**PartsUnlimited**ペインで**Edit**を選択します。
1. **PartsUnlimited** 編集ペインで、パイプラインの既存の YAML コンテンツを選択して削除し、クリップボードのコンテンツに置き換えます。
1. **PartsUnlimited** 編集ペインの右上隅にある[**Save**]をクリックし、[**Save**]ペインで[**Save**]をクリックします。これにより、このパイプラインに基づいてビルドが自動的にトリガーされます。
1. Azure DevOps ポータルの左側の垂直ナビゲーションペインの[**Pipelines**]をクリックします。
1. **Pipelines**ペインの**Recent**タブで、**PartsUnlimited**エントリをクリックし、**PartsUnlimited**ペインの**Runs**タブで、最新の実行をクリックします。 [**Summary**]ペインで、一番下までスクロールし、[**Jobs**]セクションで[**Phase 1**]をクリックして、正常に完了するまでジョブを監視します。

### 演習 2：AzureDevOps エージェントプールを管理する

この演習では、自己ホスト型の AzureDevOps エージェントを実装します。

#### タスク 1：AzureDevOps セルフホスティングエージェントを構成する

このタスクでは、LOD VM を AzureDevOps セルフホスティングエージェントとして構成し、それを使用してビルドパイプラインを実行します。

1. Lab 仮想マシン(Lab VM)またはご使用のコンピューター内で、Web ブラウザーを起動し、[Azure DevOps ポータル](https://dev.azure.com)に移動し、に関連付けられている Microsoft アカウントを使用してサインインします。
1. Azure DevOps ポータルで、Azure DevOps ページの右上隅にある[**User Settings**]アイコンをクリックし、ドロップダウンメニューで[**Personal access tokens**]をクリックします。[**Personal access tokens**]ペインをクリックし、**+ New Token**をクリックします。
1. [**Create a new personal access token**]ペインで、[**Show all scopes**]リンクをクリックし、次の設定を指定して[**Create**]をクリックします(他のすべてのスコープはデフォルト値のままにします)。

   | 設定                   | 値                                                                |
   | ---------------------- | ----------------------------------------------------------------- |
   | Name                   | **Configuring Agent Pools and Understanding Pipeline Styles lab** |
   | Scope (custom defined) | **Agent Pools**                                                   |
   | Permissions            | **Read & manage**                                                 |

1. **Success**ペインで、パーソナルアクセストークンの値をクリップボードにコピーします。

   > **注**：トークンを必ずコピーしてください。このペインを閉じると、取得できなくなります。

1. **Success**ペインで、**Close**をクリックします。
1. AzureDevOps ポータルの左上隅の**Azure DevOps**をクリックしてから、左下隅の**Organization settings**ラベルをクリックします。
1. 左側の垂直メニューの[**Pipelines**]セクションで、[**Agent pools**]をクリックします。
1. [**Agent pools**]ペインの右上隅にある[**Add pool**]をクリックします。
1. [**Add agent pool**]ペインの[**Pool type**]ドロップダウンリストで、[**Self-hosted**]を選択し、[**Name**]テキストボックスに**az400m05l05a-pool**と入力します。 次に、[**Create**]をクリックします。
1. **Agent pools**ペインに戻り、新しく作成された**az400m05l05a-pool**を表すエントリをクリックします。
1. **az400m05l05a-pool**ペインで、**New agent**ボタンをクリックします。
1. [**Get the agent**]ペインで、[**Windows**]タブと[**x64**]タブが選択されていることを確認し、[**Downloads**]をクリックして、エージェントバイナリを含む zip アーカイブをダウンロードしてダウンロードします。ユーザープロファイル内のローカルの**Downloads**フォルダーに移動します。

   > **注**：この時点で、現在のシステム設定でファイルのダウンロードができないことを示すエラーメッセージが表示された場合は、インターネットエクスプローラウィンドウの右上隅にある **設定を示す歯車の記号をクリックします。** メニューヘッダーのドロップダウンメニューで[**インターネットオプション**]を選択し、[**インターネットオプション**]ダイアログボックスで[**Details**]をクリックし、[**Details**]タブで[**リセット**]をクリックします。[**インターネットエクスプローラの設定をリセット**]ダイアログボックスで、[**リセット**]をもう一度クリックし、[**Close**]をクリックして、ダウンロードを再試行します。

1. 管理者として WindowsPowerShell を起動し、**Administrator: Windows PowerShell**コンソールから、**Get the agent**ペインに表示されている**Create the agent**コマンドをコピーします。次のスクリプトを実行して**C:\agent**ディレクトリを作成し、ダウンロードしたアーカイブのコンテンツをそのディレクトリに抽出します(すべてのコマンドが実行されていることを確認し、最後のコマンドが実行されなかった場合は[Enter]をクリックします)。これは次のようになります(最新バージョンにはコピーしたものを使用してください)。

   ```powershell
   PS C:\> mkdir agent ; cd agent
   PS C:\agent> Add-Type -AssemblyName System.IO.Compression.FileSystem ;
   [System.IO.Compression.ZipFile]::ExtractToDirectory("$HOME\Downloads\vsts-agent-win-x64-[AGENT_VERSION].zip", "$PWD")
   ```

1. **Administrator: Windows PowerShell**コンソールで、以下を実行してエージェントを構成します。

  > PowerShell ISE ではなく、Windows PowerShell コンソールの利用をお勧めします。

   ```powershell
   .\config.cmd
   ```

1. プロンプトが表示されたら、次の設定の値を指定します。

   | Setting                                                                                  | Value                                                                                                                                                                                       |
   | ---------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
   | Enter server URL                                                                         | the URL of your Azure DevOps organization, in the format **https://dev.azure.com/`<organization_name>`**, where `<organization_name>` represents the name of your Azure DevOps organization |
   | Enter authentication type (press enter for PAT)                                          | (ただエンターキーを押します)                                                                                                                                                                                   |
   | Enter personal access token                                                              | Personal access token を貼り付けます                                                                                                                                          |
   | Enter agent pool (press enter for default)                                               | **az400m05l05a-pool**                                                                                                                                                                       |
   | Enter agent name                                                                         | **az400m05-vm0**                                                                                                                                                                            |
   | Enter work folder (press enter for \_work)                                               | (ただエンターキーを押します)                                                                                                                                                                                   |
   | Enter Perform an unzip for tasks for each step. (press enter for N)                      | (ただエンターキーを押します)                                                                                                                                                                                   |
   | Enter run agent as service? (Y/N) (press enter for N)                                    | **Y**                                                                                                                                                                                       |
   | Enter User account to use for the service (press enter for NT AUTHORITY\NETWORK SERVICE) | (ただエンターキーを押します)                                                                                                                                                                                   |

   > **注**：セルフホストエージェントは、サービスまたは対話型プロセスとして実行できます。エージェントの機能の検証が簡単になるため、インタラクティブモードから始めることをお勧めします。本番環境で使用する場合は、エージェントをサービスとして実行するか、自動ログオンを有効にした対話型プロセスとして実行することを検討する必要があります。どちらも実行状態を維持し、オペレーティングシステムが再起動された場合にエージェントが自動的に起動するようにするためです。

   > **注**：エージェントが**Started successfully**ステータスを報告していることを確認してください。

1. Azure DevOps ポータルを表示しているブラウザーウィンドウに切り替えて、[Get the agent]ペインを閉じます。
1. **az400m05l05a-pool**ペインの**Agents**タブを開き、新しく構成されたエージェントが**Online**ステータスでリストされていることに注意してください。
1. Azure DevOps ポータルを表示している Web ブラウザーウィンドウの左上隅にある、**Azure DevOps**ラベルをクリックします。
1. プロジェクトのリストを表示しているブラウザウィンドウで、**Configuring Agent Pools and Understanding Pipeline Styles**プロジェクトを表すタイルをクリックします。
1. [**Configuring Agent Pools and Understanding Pipeline Styles**]ペインの左側の垂直ナビゲーションペインの[**Pipelines**]をクリックします。
1. **Pipelines**ペインの**Recent**タブで**PartsUnlimited**を選択し、**PartsUnlimited**ペインで**Edit**を選択します。
1. **PartsUnlimited**編集ペインの既存の YAML ベースのパイプラインで、行**7**の `vmImage：vs2017-win2016`を置き換えて、ターゲットエージェントプールに次のコンテンツを指定し、新しく作成されたセルフホステッドエージェントプールを指定します：

   ```yaml
   pool:
     name: az400m05l05a-pool
     demands:
       - agent.name -equals az400m05-vm0
   ```

1. [`Task: NugetInstaller@0`]の上にある**Settings(タスクの上に灰色で表示されているリンク)** をクリックし、**[Advanced]> [NuGet Version]> 4.0.0**を変更して、[**add**]をクリックします。
1. **PartsUnlimited** Edit ペインのペインの右上隅にある[**Save**]をクリックし、[**Save**]ペインでもう一度[**Save**]をクリックします。これにより、このパイプラインに基づいてビルドが自動的にトリガーされます。
1. Azure DevOps ポータルの左側の垂直ナビゲーションペインの[**Pipelines**]セクションで、[**Pipelines**]をクリックします。
1. **Pipelines**ペインの**Recent**タブで、**PartsUnlimited**エントリをクリックし、**PartsUnlimited**ペインの**Runs**タブで、最新の実行を選択します。[**Summary**]ペインで、一番下までスクロールし、[**Jobs**]セクションで[**Phase 1**]をクリックして、正常に完了するまでジョブを監視します。

#### レビュー

このラボでは、従来のパイプラインを YAML ベースのパイプラインに変換する方法と、セルフホストエージェントを実装して使用する方法を学習しました。
