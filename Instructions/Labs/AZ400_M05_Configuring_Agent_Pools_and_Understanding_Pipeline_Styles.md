---
lab:
    title: 'ラボ: エージェント プールの構成とパイプライン スタイルの理解'
    module: 'モジュール 5: Configuring Azureの構成'
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

-   推定時間: **90 分**

## 指示

### 開始する前に

#### ラボの仮想マシンにログインする

次の資格情報を使用して、Windows 10 コンピューターにサインインしていることを確認します。
    
-   ユーザー名: **Student**
-   パスワード: **Pa55w.rd**

#### インストールされているアプリケーションを確認します

Windows 10 デスクトップでタスク バーを探します。タスク バーには、この課題で使用するアプリケーションのアイコンが含まれています:
    
-   Microsoft Edge
-   [Visual Studio Code](https://code.visualstudio.com/)。これは、このラボの前提条件の一部としてインストールされます。

#### Azure DevOps 組織を設定する

このラボで使用できる Azure DevOps 組織がまだない場合は、[組織またはプロジェクト コレクションの作成](https://docs.microsoft.com/ja-jp/azure/devops/organizations/accounts/create-organization?view=azure-devops)の手順に従って作成してください。

#### Azure サブスクリプションを準備する

-   既存の Azure サブスクリプションを特定するか、新しいサブスクリプションを作成します。
-   Azure サブスクリプションの所有者のロールと Azure サブスクリプションに関連付けられた Azure AD テナントのグローバル管理者ロールを持つ Microsoft アカウントまたは Azure AD アカウントを持っていることを確認します。詳細については、[Azure ポータルを使用した Azure のロールno割り当ての一覧表示](https://docs.microsoft.com/ja-jp/azure/role-based-access-control/role-assignments-list-portal) および[Azure Active Directory での管理者のロールの表示と割り当て](https://docs.microsoft.com/ja-jp/azure/active-directory/roles/manage-roles-portal#view-my-roles) を参照してください。

#### GitHub アカウントを設定する

[新しい GitHub アカウントにサインアップする](https://docs.github.com/en/free-pro-team@latest/github/getting-started-with-github/signing-up-for-a-new-github-account) にある手順に従ってください。

### 演習 0: ラボの前提条件を構成する

この演習では、ラボの前提条件を設定します。Azure DevOps Demo Generator テンプレートに基づいて事前構成された Parts Unlimited チームプロジェクトで構成されます。

#### タスク 1: チーム プロジェクトを構成する

このタスクでは、Azure DevOps Demo Generator を使用して、**PartsUnlimited** テンプレートに基づいて新しいプロジェクトを生成します。

1.  ラボ コンピューターで、Web ブラウザーを起動し、[Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net) に移動します。このユーティリティ サイトは、ラボに必要なコンテンツ (作業項目、リポジトリなど) が事前に入力されたアカウント内に新しい Azure DevOps プロジェクトを作成するプロセスを自動化します。 

    > **注**: サイトの詳細については、https://docs.microsoft.com/ja-jp/azure/devops/demo-gen を参照してください。

1.  「**サインイン**」 をクリックし、Azure DevOps サブスクリプションに関連付けられている Microsoft アカウントを使用してサインインします。
1.  必要な場合は、**Azure DevOps Demo Generator** ページで、「**承諾する**」 をクリックして、Azure DevOps サブスクリプションにアクセスするためのアクセス許可の要求を受け入れます。
1.  「**新しいプロジェクトの作成**」 ページの 「**新しいプロジェクト名**」 テキスト ボックスに「**エージェント プールの構成とパイプライン スタイルの理解**」と入力し、「**組織の選択**」 ドロップダウン リストで、Azure DevOps 組織を選択して、「**テンプレートの選択**」 をクリックします。
1.  「**テンプレートの選択**」 ページで、**PartsUnlimited** テンプレートをクリックし、「**テンプレートの選択**」 をクリックします。
1.  「**新しいプロジェクトの作成**」 ページに戻り、「**ARM 出力**」 ラベルの下のチェックボックスを選択して、「**プロジェクトの作成**」 をクリックします。

    > **注**: プロセスが完了するまでお待ちください。これにはおよそ 2 分かかります。プロセスが失敗した場合は、DevOps 組織に移動し、プロジェクトを削除して、再試行してください。

1.  「**新規プロジェクトの作成**」 ページで、「**プロジェクトに移動**」 をクリックします

#### タスク 2: Visual Studio Code をインストールする

このタスクでは、Visual StudioCode をインストールします。これらの前提条件をすでに実装している場合は、次のタスクに直接進むことができます。

1.  Visual Studio Code をまだインストールしていない場合は、Web ブラウザー ウィンドウから [Visual Studio Code ダウンロード ページ](https://code.visualstudio.com/)に移動し、ダウンロードしてインストールします。 

#### タスク 3: イメージ デプロイ用のラボ環境を準備する

このタスクでは、セルフホスティド エージェントのプロビジョニングに使用されるイメージのデプロイ用にラボ環境を準備します。

> **注**: セルフホスティドティド エージェントの場所とその構成は、要件によって大きく異なります。この演習では、この目的のために、Microsoft がホストするエージェントに使用されるのと同じイメージに基づく Azure VM を使用します。イメージは、[仮想環境 GitHub リポジトリ](https://github.com/actions/virtual-environments)から入手できます。また、GitHub Actions でホストされているランナーの画像もあります。

1.  ラボ コンピューターで、Web ブラウザーを起動し、[GitHub](https://github.com) に移動して、GitHub アカウントにサインインします。
1.  GitHub アカウントを表示している Web ブラウザーで、右上隅にあるプロファイルを表す丸いアイコンをクリックし、ドロップダウン メニューで 「**設定**」 をクリックします。
1.  「**パブリック プロファイル**」 ページの左側の垂直メニューで、「**開発者設定**」 をクリックします。
1.  「**GitHub アプリ**」 ページの左側の垂直メニューで、「**パーソナル アクセス トークン**」 をクリックします。
1.  「**パーソナル アクセス トークン**」 ページの右上隅にある 「**新しいトークンの生成**」 をクリックします。
1.  「**新しいパーソナル アクセス トークン**」 ページの 「**メモ**」 テキスト ボックスに「**az400m05l05alab**」と入力し、「**スコープの選択**」 セクションで 「**read:packages**」 チェックボックスを選択して、「**トークンの生成**」 をクリックします。
1.  「**パーソナル アクセス トークン**」 ページで、新しく生成されたトークンを特定し、その値を記録します。このタスクの後半で必要になります。

    > **注**: この時点で、必ずパーソナル アクセス トークンを記録してください。現在のページから移動すると、その値を取得できなくなります。 

1.  ラボ コンピューターで、Web ブラウザーを起動し、[Packer ダウンロード ページ](https://www.packer.io/downloads)に移動し、現在のバージョンの Windows 64 ビットバージョンの Packer をダウンロードし、**packer.exe** バイナリを含むアーカイブを開いて、**C:\Windows** ディレクトリに抽出します。 
1.  ラボ コンピューターで、管理者として Windows PowerShell ISE を起動し、「**管理者: Windows PowerShell ISE**」 ウィンドウのコンソール ペインから、次を実行して Chocolatey をインストールします。

    ```powershell
    Set-ExecutionPolicy Bypass -Scope Process -Force; 「System.Net.ServicePointManager」::SecurityProtocol = 「System.Net.ServicePointManager」::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
    ```

1.  「**管理者: Windows PowerShell ISE**」 ウィンドウのコンソール ペインから、次を実行して git インストールします。

    ```powershell
    choco install git -params '"/GitOnlyOnPath"' -y
    ```

1.  「**管理者: Windows PowerShell ISE**」 ウィンドウで、次を実行して packer をインストールします。

    ```powershell
    choco install packer -y
    ```

1.  「**管理者: Windows PowerShell ISE**」 ウィンドウのコンソール ペインから、次を実行して Azure CLI インストールします。

    ```powershell
    Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'; rm .\AzureCLI.msi
    ```

1.  「**管理者: Windows PowerShell ISE**」 ウィンドウのコンソール ペインから、次を実行して PowerShell インストールします (続行するための確認を求められたら、「すべてはい」をクリックします)。

    ```powershell
    Install-Module -Name Az
    ```

1.  「**管理者: Windows PowerShell ISE**」 ウィンドウのコンソール ペインから、次のコマンドを実行して、Azure サブスクリプションにサインインします (資格情報の入力を求められたら、Azure サブスクリプションの所有者のロールと Azure サブスクリプションに関連付けられた Azure AD テナントのグローバル管理者のロールを持つ Microsoft アカウントまたは Azure AD アカウントで認証します)。

    ```powershell
    Add-AzAccount
    ```

1.  「**管理者: Windows PowerShell ISE**」 ウィンドウのコンソール ペインから、次を実行して git インストールします。

    ```powershell
    New-Item -Type Directory -Path 'C:\Labfiles' -Force
    Set-Location c:\Labfiles
    git clone https://github.com/actions/virtual-environments.git virtual-environments -q
    ```

1.  ラボ コンピューターで、ファイル エクスプローラーを起動し、**C:\\Labfiles\\virtual-environments\\images\\win** ディレクトリに移動して、メモ帳で **windows2019.json** ファイルを開き、その内容を確認します。

    > **注**: このファイルは、イメージの内容を定義します。**プロビジョナー** セクションを変更することで、構成をカスタマイズし、効果的にイメージ プロビジョニング時間に影響を与えることができます。

1.  ラボ コンピューターのファイル エクスプローラーで、**C:\\Labfiles\\virtual-environments\\helpers** ディレクトリに移動し、メモ帳で **GenerateResourcesAndImage.ps1** ファイルを開きます。 
1.  メモ帳で **GenerateResourcesAndImage.ps1** ファイルの内容を表示しながら、コマンド `New-AzStorageAccount -ResourceGroupName $ResourceGroupName -AccountName $storageAccountName -Location $AzureLocation -SkuName "Standard_LRS"`` を含む行で、`"Standard_LRS"` を `"Premium_LRS"` に置き替え、変更を保存し、ファイルを閉じます。

    > **注**: この変更は、イメージのプロビジョニングを高速化することを目的としています。

1.  ラボ コンピューターのファイル エクスプローラーで、**C:\\Labfiles\\virtual-environments\\images\\win** ディレクトリに移動し、メモ帳で **windows2016.json** ファイルを開き、次のコンテンツを貼り付け、既存のコンテンツを置き換え、変更を保存して、ファイルを閉じます。

    > **注**: これらの変更は、この特定のラボシナリオでのイメージ プロビジョニングを高速化することを厳密に目的としています。一般に、要件に合わせてイメージを調整する必要があります。 

    ```json
    {
        "variables": {
            "client_id": "{{env `ARM_CLIENT_ID`}}",
            "client_secret": "{{env `ARM_CLIENT_SECRET`}}",
            "subscription_id": "{{env `ARM_SUBSCRIPTION_ID`}}",
            "tenant_id": "{{env `ARM_TENANT_ID`}}",
            "object_id": "{{env `ARM_OBJECT_ID`}}",
            "resource_group": "{{env `ARM_RESOURCE_GROUP`}}",
            "storage_account": "{{env `ARM_STORAGE_ACCOUNT`}}",
            "temp_resource_group_name": "{{env `TEMP_RESOURCE_GROUP_NAME`}}",
            "location": "{{env `ARM_RESOURCE_LOCATION`}}",
            "virtual_network_name": "{{env `VNET_NAME`}}",
            "virtual_network_resource_group_name": "{{env `VNET_RESOURCE_GROUP`}}",
            "virtual_network_subnet_name": "{{env `VNET_SUBNET`}}",
            "private_virtual_network_with_public_ip": "{{env `PRIVATE_VIRTUAL_NETWORK_WITH_PUBLIC_IP`}}",
            "vm_size": "Standard_D8s_v3",
            "run_scan_antivirus": "false",
            "root_folder": "C:",
            "toolset_json_path": "{{env `TEMP`}}\\toolset.json",
            "image_folder": "C:\\image",
            "imagedata_file": "C:\\imagedata.json",
            "helper_script_folder": "C:\\Program Files\\WindowsPowerShell\\Modules\\",
            "psmodules_root_folder": "C:\\Modules",
            "install_user": "installer",
            "install_password": null,
            "capture_name_prefix": "packer",
            "image_version": "dev",
            "image_os": "win16"
        },
        "sensitive-variables": [
            "install_password",
            "client_secret"
        ],
        "builders": [
            {
                "name": "vhd",
                "type": "azure-arm",
                "client_id": "{{user `client_id`}}",
                "client_secret": "{{user `client_secret`}}",
                "subscription_id": "{{user `subscription_id`}}",
                "object_id": "{{user `object_id`}}",
                "tenant_id": "{{user `tenant_id`}}",
                "os_disk_size_gb": "256",
                "location": "{{user `location`}}",
                "vm_size": "{{user `vm_size`}}",
                "resource_group_name": "{{user `resource_group`}}",
                "storage_account": "{{user `storage_account`}}",
                "temp_resource_group_name": "{{user `temp_resource_group_name`}}",
                "capture_container_name": "images",
                "capture_name_prefix": "{{user `capture_name_prefix`}}",
                "virtual_network_name": "{{user `virtual_network_name`}}",
                "virtual_network_resource_group_name": "{{user `virtual_network_resource_group_name`}}",
                "virtual_network_subnet_name": "{{user `virtual_network_subnet_name`}}",
                "private_virtual_network_with_public_ip": "{{user `private_virtual_network_with_public_ip`}}",
                "os_type": "Windows",
                "image_publisher": "MicrosoftWindowsServer",
                "image_offer": "WindowsServer",
                "image_sku": "2016-Datacenter",
                "communicator": "winrm",
                "winrm_use_ssl": "true",
                "winrm_insecure": "true",
                "winrm_username": "packer"
            }
        ],
        "provisioners": [
            {
                "type": "powershell",
                "inline": [
                    "New-Item -Path {{user `image_folder`}} -ItemType Directory -Force"
                ]
            },
            {
                "type": "file",
                "source": "{{ template_dir }}/scripts/ImageHelpers",
                "destination": "{{user `helper_script_folder`}}"
            },
            {
                "type": "file",
                "source": "{{ template_dir }}/scripts/SoftwareReport",
                "destination": "{{user `image_folder`}}"
            },
            {
                "type": "file",
                "source": "{{ template_dir }}/post-generation",
                "destination": "C:/"
            },
            {
                "type": "file",
                "source": "{{ template_dir }}/scripts/Tests",
                "destination": "{{user `image_folder`}}"
            },
            {
                "type": "file",
                "source": "{{template_dir}}/toolsets/toolset-2016.json",
                "destination": "{{user `toolset_json_path`}}"
            },
            {
                "type": "windows-shell",
                "inline": [
                    "net user {{user `install_user`}} {{user `install_password`}} /add /passwordchg:no /passwordreq:yes /active:yes /Y",
                    "net localgroup Administrators {{user `install_user`}} /add",
                    "winrm set winrm/config/service/auth @{Basic=\"true\"}",
                    "winrm get winrm/config/service/auth"
                ]
            },
            {
                "type": "powershell",
                "inline": [
                    "if (-not ((net localgroup Administrators) -contains '{{user `install_user`}}')) { exit 1 }"
                ]
            },
            {
                "type": "powershell",
                "environment_vars": [
                    "ImageVersion={{user `image_version`}}",
                    "TOOLSET_JSON_PATH={{user `toolset_json_path`}}",
                    "PSMODULES_ROOT_FOLDER={{user `psmodules_root_folder`}}"
                ],
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Install-PowerShellModules.ps1",
                    "{{ template_dir }}/scripts/Installers/Initialize-VM.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-WebPlatformInstaller.ps1"
                ],
                "execution_policy": "unrestricted"
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Update-DotnetTLS.ps1"
                ]
            },
            {
                "type": "windows-restart",
                "restart_timeout": "30m"
            },
            {
                "type": "powershell",
                "inline": [
                    "setx ImageVersion {{user `image_version` }} /m",
                    "setx ImageOS {{user `image_os` }} /m"
                ]
            },
            {
                "type": "powershell",
                "environment_vars": [
                    "IMAGE_VERSION={{user `image_version`}}",
                    "IMAGEDATA_FILE={{user `imagedata_file`}}"
                ],
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Update-ImageData.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-PowershellCore.ps1"
                ]
            },
            {
                "type": "windows-restart",
                "restart_timeout": "30m"
            },
            {
                "type": "powershell",
                "valid_exit_codes": [
                    0,
                    3010
                ],
                "environment_vars": [
                    "TOOLSET_JSON_PATH={{user `toolset_json_path`}}"
                ],
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Install-VS.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-NET48.ps1"
                ],
                "elevated_user": "{{user `install_user`}}",
                "elevated_password": "{{user `install_password`}}"
            },
                {
                "type": "powershell",
                "environment_vars": [
                    "TOOLSET_JSON_PATH={{user `toolset_json_path`}}"
                ],
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Install-Nuget.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-Vsix.ps1"
                ]
            },
            {
                "type": "windows-restart",
                "restart_timeout": "30m"
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Install-NodeLts.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-7zip.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-Packer.ps1"
                ]
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Install-OpenSSL.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-Git.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-GitHub-CLI.ps1"
                ]
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Enable-DeveloperMode.ps1"
                ],
                "elevated_user": "{{user `install_user`}}",
                "elevated_password": "{{user `install_password`}}"
            },
            {
                "type": "windows-shell",
                "inline": [
                    "wmic product where \"name like '%%microsoft azure powershell%%'\" call uninstall /nointeractive"
                ]
            },
            {
                "type": "powershell",
                "environment_vars": [
                    "TOOLSET_JSON_PATH={{user `toolset_json_path`}}"
                ],
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Install-Cmake.ps1"
                ]
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Install-GitVersion.ps1"
                ]
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Configure-DynamicPort.ps1"
                ],
                "elevated_user": "{{user `install_user`}}",
                "elevated_password": "{{user `install_password`}}"
            },
            {
                "type": "windows-restart",
                "restart_timeout": "30m"
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Finalize-VM.ps1"
                ]
            },
            {
                "type": "windows-restart",
                "restart_timeout": "30m"
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Configure-Antivirus.ps1",
                    "{{ template_dir }}/scripts/Installers/Disable-JITDebugger.ps1"
                ]
            },
            {
                "type": "powershell",
                "inline": [
                    "if( Test-Path $Env:SystemRoot\\System32\\Sysprep\\unattend.xml ){ rm $Env:SystemRoot\\System32\\Sysprep\\unattend.xml -Force}",
                    "& $env:SystemRoot\\System32\\Sysprep\\Sysprep.exe /oobe /generalize /quiet /quit",
                    "while($true) { $imageState = Get-ItemProperty HKLM:\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Setup\\State | Select ImageState; if($imageState.ImageState -ne 'IMAGE_STATE_GENERALIZE_RESEAL_TO_OOBE') { Write-Output $imageState.ImageState; Start-Sleep -s 10  } else { break } }"
                ]
            }
        ]
    }
    ```

1.  ラボ　コンピューターのファイルエクスプローラーで、**C:\\Labfiles\\virtual-environments\\images\\win\\scripts\\Installers** ディレクトリに移動し、メモ帳で **VisualStudio.Tests.ps1** ファイルを開き、次のセクションを見つけます。

    ```powershell
    $workLoads = @(
	"--allWorkloads --includeRecommended"
	$requiredComponents
	"--remove Component.CPython3.x64"
    )
    ```

1.  **VisualStudio.Tests.ps1** ファイルの内容を表示するメモ帳ウィンドウで、前の手順で特定したセクションを次の内容に置き換えます。

    ```powershell
    $workLoads = @(
	"--add Microsoft.VisualStudio.Workload.NetWeb --add Component.GitHub.VisualStudio --includeRecommended"
	"--remove Component.CPython3.x64"
    )
    ```

    > **注**: この変更は、この特定のラボ シナリオでのイメージ プロビジョニングを高速化することを厳密に目的としています。一般に、要件に合わせてインストールする Visual Studio コンポーネントを調整する必要があります。 

1.  ラボ コンピューターのファイル エクスプローラーで、**C:\\Labfiles\\virtual-environments\\images\\win\\scripts\\Tests** ディレクトリに移動し、メモ帳で **VisualStudio.Tests.ps1** ファイルを開き、`$expectedComponents = Get-ToolsetContent | Select-Object -ExpandProperty visualStudio | Select-Object -ExpandProperty workloads` の行を `$expectedComponents = "Microsoft.VisualStudio.Workload.NetWeb"` に置き替え、変更を保存して、ファイルを閉じます。

    > **注**: この変更は、この特定のラボ シナリオでのイメージ プロビジョニングを高速化することを厳密に目的としています。一般に、要件に一致するようにテストを調整する必要があります。 

1.  「**管理者: Windows PowerShell ISE**」 ウィンドウに切り替え、コンソール ペインから、次のコマンドを実行し、セルフホストティド エージェントのプロビジョニングに使用するオペレーティング システムイメージの生成に使用するモジュールをインポートします。

    ```powershell
    Set-Location c:\Labfiles\virtual-environments
    Import-Module .\helpers\GenerateResourcesAndImage.ps1
    ```

1.  「**管理者: Windows PowerShell ISE**」 ウィンドウのスクリプト ペインから、以下を実行して、Packer PowerShell モジュールを使用し、セルフホスティド エージェントのプロビジョニングに使用するオペレーティング システム イメージを生成します (`<Azure_region>` プレースホルダーを Azure VM をデプロイする Azure リージョンの名前に置き換え、`<GitHub_PAT>` プレースホルダーをこのタスクの前半で生成した GitHub パーソナル アクセス トークンの値に置き換えます)。

    ```powershell
    $location = '<Azure_region>'
    $githubPAT = '<GitHub_PAT>'
    $random = (-join (((48..57)+(65..90)+(97..122)) * 80 | Get-Random -Count 6 |%{「char」$_})).ToLower()
    $rgName = "az400m05$random-RG"
    $subscriptionId = (Get-AzSubscription).SubscriptionId  
    GenerateResourcesAndImage -SubscriptionId $subscriptionId -ResourceGroupName $rgName -ImageGenerationRepositoryRoot "$pwd" -ImageType 'Windows2016' -AzureLocation $location -GitHubFeedToken $githubPAT
    ```

1.  プロンプトが表示されたら、Azure サブスクリプションに関連付けられている Azure AD テナントへの認証に以前に使用したのと同じユーザー アカウントでサインインします。

    > **注**: スクリプトが完了するのを待たずに、次の演習に進んでください。イメージのプロビジョニングには約 60 分かかる場合があります。

### 演習 1: YAML ベースの Azure DevOps パイプラインを作成する

この演習では、従来の Azure DevOps パイプラインを YAML ベースのパイプラインに変換します。 

#### タスク 1: Azure DevOps YAML パイプラインを作成する

このタスクでは、テンプレートベースの Azure DevOps YAML パイプラインを作成します。

1.  **エージェント プールの構成とパイプライン スタイルの理解**プロジェクトが開いた状態で Azure DevOps ポータルを表示している Web ブラウザーから、左側の垂直ナビゲーション ペインで 「**パイプライン**」 をクリックします。 
1.  「**パイプライン**」 ペインの 「**最近**」 タブで、「**新しいパイプライン**」 をクリックします。
1.  「**コードはどこにありますか?**」 ペインで、「**Azure Repos Git**」 をクリックします。 
1.  「**リポジトリの選択**」 ペインで、「**PartsUnlimited**」 をクリックします。
1.  「**パイプラインの構成**」 ペインで、「**スターター パイプライン**」 をクリックします。
1.  「**パイプライン YAML の確認**」 ペインでサンプル パイプラインを確認し、「**保存して実行**」 ボタンの横にある下向きのキャレット記号をクリックして 「**保存**」 をクリックし、「**保存**」 ペインで 「**保存**」 をクリックします。

    > **注**: これにより、プロジェクト コードをホストするリポジトリのルート ディレクトリに **azure-pipelines.yml** ファイルが作成されます。

    > **注**: サンプル YAML パイプラインのコンテンツをクラシック エディターによって生成されたパイプラインのコードに置き換え、クラシック パイプラインと YAML パイプラインの違いを考慮して変更します。

#### タスク 2: クラシック パイプラインを YAML パイプラインに変換する

このタスクでは、クラシック パイプラインを YAML パイプラインに変換します。

1.  **エージェント プールの構成とパイプライン スタイルの理解**プロジェクトが開いた状態で Azure DevOps ポータルを表示している Web ブラウザーから、左側の垂直ナビゲーション ペインで 「**パイプライン**」 をクリックします。 
1.  「**パイプライン**」 ペインの 「**最近**」 タブで、**PartsUnlimitedE2E** エントリを含むエントリの右端にマウス ポインターを合わせると、「**その他**」 メニューを示す縦の省略記号が表示され、省略記号をクリックして、ドロップダウン メニューで 「**編集**」 をクリックします。これにより、ラボの開始時に生成したプロジェクトの一部であるビルド パイプラインが表示されます。 
1.  **PartsUnlimitedE2E** 編集ペインの 「**タスク**」 タブで、「**トリガー**」 をクリックし、**PartsUnlimited** ペインの右側で 「**継続的インテグレーションを有効にする**」 チェックボックスをオフにし、「**保存とキュー**」 ボタンの横にある下向きのキャレットをクリックし、ドロップダウン メニューで 「**保存**」 をクリックし、「**ビルド パイプラインの保存**」 で 「**保存**」 をクリックします。

    > **注**: これにより、このラボ中にリポジトリが変更されたために、意図しない自動ビルドが実行されるのを防ぐことができます。

    > **注**: または、コンテンツを新しいパイプラインにコピーしたら、既存のパイプラインを削除することもできます。

1.  Azure DevOps ポータルの左側の垂直ナビゲーション ペインの 「**パイプライン**」 セクションで、「**パイプライン**」 をクリックします。 
1.  「**パイプライン**」 ペインの 「**最近**」 タブで、**PartsUnlimitedE2E** エントリをクリックします。 
1.  **PartsUnlimitedE2E** ペインの 「**実行**」 タブの右上隅にある縦の省略記号をクリックし、ドロップダウン メニューで 「**YAML にエクスポート**」 をクリックします。これにより、**build.yml** ファイルがローカルの**ダウンロード** フォルダーに自動的にダウンロードされます。

    > **注**: 「**YAML にエクスポート**」 機能は、Azure DevOp sポータル内のパイプライン エディター ペインから使用できる古い 「**YAML の表示**」 オプションに置き換わるもので、一度に 1 つのジョブで YAML コンテンツを表示するように制限されていました。新しい機能は、YAML 解析ライブラリを含む既存のクラシックおよび YAML パイプライン インフラストラクチャを活用し、2 つの間のより正確な変換を実現します。次のパイプライン コンポーネントをサポートします。

    - 単一および複数のジョブ
    - チェックアウト オプション
    - 実行プランの並列処理
    - タイムアウトと名前のメタデータ
    - 需要
    - スケジュールとその他のトリガー
    - デフォルトとは異なるジョブを含むプールの選択
    - すべてのタスクとすべての入力 (デフォルトの入力の最適化を含む)
    - ジョブとステップの条件
    - タスク グループの展開

    > **注**: 新しい機能でカバーされていないコンポーネントは、変数とタイムゾーン変換のみです。YAML で定義された変数は、Azure DevOps ポータルのユーザー インターフェイスで設定された変数をオーバーライドします。**YAML へのエクスポート**機能がクラシック パイプライン変数の存在を検出した場合、それらは新しく生成された YAML パイプライン定義内のコメントに明示的に含まれます。同様に、YAML の cron スケジュールは UTC で表されるため、従来のスケジュールは組織のタイムゾーンに依存するため、その存在もコメントに含まれます。

    > **注**: この機能の詳細については、[View YAML」の置き換え](https://devblogs.microsoft.com/devops/replacing-view-yaml/)を参照してください。

1.  ラボ コンピューターで、Visual Studio Code を起動し、それを使用してファイル **build.yml** を開きます。ファイルには、次の内容を含める必要があります。

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
          Contents: '**/*.json'
          TargetFolder: $(build.artifactstagingdirectory)
      - task: PublishBuildArtifacts@1
        name: PublishBuildArtifacts_5
        displayName: Publish Artifact
        inputs:
          PathtoPublish: $(build.artifactstagingdirectory)
          TargetPath: '\\my\share\$(Build.DefinitionName)\$(Build.BuildNumber)'
    ...
    ```

1.  Visual Studio Code ウィンドウのトップレベル メニューで 「**選択**」 をクリックし、ドロップダウン メニューで 「**すべて選択**」 をクリックします。
1.  Visual Studio Code ウィンドウのトップレベルメニューで 「**編集**」 をクリックし、ドロップダウン メニューで 「**コピー**」 をクリックします。
1.  Azure DevOps ポータルを表示しているブラウザー ウィンドウに切り替え、左側の垂直ナビゲーション ペインの 「**パイプライン**」 セクションで、「**パイプライン**」 をクリックします。 
1.  「**パイプライン**」 ペインの 「**最近**」 タブで 「**PartsUnlimited**」 を選択し、「**PartsUnlimited**」 ペインで 「**編集**」 を選択します。
1.  **PartsUnlimited** 編集ペインで、パイプラインの既存の YAML コンテンツを選択し、クリップボードのコンテンツに置き換えます。
1.  **PartsUnlimited** 編集ペインの右上隅にある 「**保存**」 をクリックし、「**保存**」 ペインの 「**保存**」 をクリックします。これにより、このパイプラインに基づいてビルドが自動的にトリガーされます。 
1.  Azure DevOps ポータルの左側の垂直ナビゲーション ペインの 「**パイプライン**」 セクションで、「**パイプライン**」 をクリックします。
1.  「**パイプライン**」 ペインの 「**最近**」 タブで、「**PartsUnlimited**」 エントリをクリックし、「**PartsUnlimited**」 ペインの 「**実行**」 タブで最新の実行を選択し、実行の 「**概要**」 ペインで一番下までスクロールし、「**ジョブ**」 セクションで 「**フェーズ 1**」 をクリックし、正常に完了するまでジョブを監視します。 

### 演習 2: Azure DevOps エージェント プールを管理する

この演習では、セルフホスティド Azure DevOps エージェントを実装します。

> **注**: この演習を開始する前に、ラボの最初に開始したイメージ プロビジョニング スクリプトが正常に完了していることを確認してください。

#### タスク 1: Azure VM ベースのセルフホスティド エージェントをデプロイする

このタスクでは、このラボの最初に生成したカスタム イメージを使用して、セルフホスティド エージェントをプロビジョニングするために使用される Azure VMをデプロイします。

1.  ラボ コンピューターで、「**管理者: Windows PowerShell ISE**」 ウィンドウに切り替え、このラボの最初に呼び出した **GenerateResourcesAndImage** スクリプトの出力を確認し、生成された vhd ファイルへのフルパスを表す **OSDiskUri** 文字列の最初の部分を調べてストレージ アカウント名の値を特定します。

    > **注**: 出力は、次の値 `OSDiskUri：https：//az400m05l05xrg001.blob.core.windows.net/system/Microsoft.Compute/Images/images/packer-osDisk.40096600-7cac-47f4-8573-ffaa113c78b1のようになります。 v
hd' のようになります。ここで、`az400m05l05xrg001` は、ストレージ アカウント名を表します。

1.  ラボ コンピューターから Web ブラウザーを起動し、[**Azure Portal**](https://portal.azure.com) に移動して、前のタスクで Azure VM のプロビジョニングに使用したユーザー アカウントでサインインします。 
1.  Azure portal で、**ストレージ アカウント**のリソースの種類を検索して選択し、「**ストレージ アカウント**」 ブレードで、このタスクの前半で特定したストレージ アカウントの名前をクリックします。 
1.  ストレージ アカウント ブレードで、ストレージ アカウントを含むリソース グループの名前をメモします。このタスクの後半で必要になります。
1.  ストレージ アカウント ブレードの左側の垂直メニューの 「**Blob サービス**」 セクションで、「**コンテナー**」 をクリックします。
1.  「コンテナー」 ブレードで 「**+ コンテナー**」 をクリックし、「**新しいコンテナー**」 ブレードで 「**名前**」 テキスト ボックスに「**disks**」と入力し、「**パブリック アクセス レベル**」 ドロップボックスに**プライベート (匿名アクセスなし)** エントリが含まれていることを確認して、「**作成**」 をクリックします。
1.  「コンテナー」 ブレードに戻り、コンテナーのリストで 「**イメージ**」 をクリックし、「**イメージ**」 ブレードで、新しく生成されたイメージを表すエントリをクリックします。
1.  イメージのプロパティを表示しているブレードで、**URL** エントリの横にある 「**クリップボードにコピー**」 アイコンをクリックし、コピーした値をメモ帳に貼り付けます。これは、次のステップで必要になります。 
1.  ラボ コンピューターで、「**管理者: Windows PowerShell ISE**」 ウィンドウに切り替え、スクリプト ペインから次のコマンドを実行して、Azure Resource Manager テンプレートを使用して Azure VM のデプロイに必要な変数を設定します (`<resource_group_name>` プレースホルダーを、このタスクの前半で特定したリソース グループの名前に置き換えます。`<storage_account_name>` プレースホルダーを、このタスクの前半で特定したストレージ アカウントの名前に置き換えます。`<image_url>` プレースホルダーを、このタスクの前半で特定したイメージ URLの名前に置き換えます)。

    ```powershell
    $rgName = `<resource_group_name>'
    $storageAccountName = '<storage_account_name>'
    $imageUrl = '<image_url>'
    ```

1.  ラボ　コンピューターで、「ファイル クスプローラー」 ウィンドウに切り替え、**C:\\Labfiles** フォルダーに **az400m05-vm0.deployment.template.json** という名前の新しいファイルを次の内容で作成し、保存して閉じます。 

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
          "vmSize": {
            "type": "string",
            "defaultValue": "Standard_D2s_v3",
            "metadata": {
              "description": "Virtual machine size"
            }
          },
          "adminUsername": {
            "type": "string",
            "metadata": {
              "description": "Admin username"
            }
          },
          "adminPassword": {
            "type": "securestring",
            "metadata": {
              "description": "Admin password"
            }
          },
          "storageAccountName": {
            "type": "string",
            "metadata": {
               "description": "storage account name"
            }
          },
          "imageUrl": {
            "type": "string",
            "metadata": {
               "description": "image Url"
            }
          },
          "diskContainer": {
            "type": "string",
            "defaultValue": "disks",
            "metadata": {
              "description": "disk container"
            }
          }
        },
      "variables": {
        "vmName": "az400m05-vm0",
        "osType": "Windows",
        "osDiskVhdName": "[concat('http://',parameters('storageAccountName'),'.blob.core.windows.net/',parameters('diskContainer'),'/',variables('vmName'),'-osDisk.vhd')]",
        "nicName": "az400m05-vm0-nic0",
        "virtualNetworkName": "az400m05-vnet0",
        "publicIPAddressName": "az400m05-vm0-nic0-pip",
        "nsgName": "az400m05-vm0-nic0-nsg",
        "vnetIpPrefix": "10.0.0.0/22", 
        "subnetIpPrefix": "10.0.0.0/24", 
        "subnetName": "subnet0",
        "subnetRef": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('virtualNetworkName'), variables('subnetName'))]",
        "computeApiVersion": "2018-06-01",
        "networkApiVersion": "2018-08-01"
      },
        "resources": [
            {
                "name": "[variables('vmName')]",
                "type": "Microsoft.Compute/virtualMachines",
                "apiVersion": "[variables('computeApiVersion')]",
                "location": "[resourceGroup().location]",
                "dependsOn": [
                    "[variables('nicName')]"
                ],
                "properties": {
                    "osProfile": {
                        "computerName": "[variables('vmName')]",
                        "adminUsername": "[parameters('adminUsername')]",
                        "adminPassword": "[parameters('adminPassword')]",
                        "windowsConfiguration": {
                            "provisionVmAgent": "true"
                        }
                    },
                    "hardwareProfile": {
                        "vmSize": "[parameters('vmSize')]"
                    },
                    "storageProfile": {
                        "osDisk": {
                            "name": "[concat(variables('vmName'),'-osDisk')]",
                            "osType": "[variables('osType')]",
                            "caching": "ReadWrite",
                            "image": {
                                "uri": "[parameters('imageUrl')]"
                            },
                            "vhd": {
                                "uri": "[variables('osDiskVhdName')]"
                            },
                            "createOption": "fromImage"
                        },
                        "dataDisks": []
                    },
                    "networkProfile": {
                        "networkInterfaces": [
                            {
                                "properties": {
                                    "primary": true
                                },
                                "id": "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
                            }
                        ]
                    }
                }
            },
            {
                "type": "Microsoft.Network/virtualNetworks",
                "name": "[variables('virtualNetworkName')]",
                "apiVersion": "[variables('networkApiVersion')]",
                "location": "[resourceGroup().location]",
                "comments": "Virtual Network",
                "properties": {
                    "addressSpace": {
                        "addressPrefixes": [
                            "[variables('vnetIpPrefix')]"
                        ]
                    },
                    "subnets": [
                        {
                            "name": "[variables('subnetName')]",
                            "properties": {
                                "addressPrefix": "[variables('subnetIpPrefix')]"
                            }
                        }
                    ]
                }
            },
            {
                "name": "[variables('nicName')]",
                "type": "Microsoft.Network/networkInterfaces",
                "apiVersion": "[variables('networkApiVersion')]",
                "location": "[resourceGroup().location]",
                "comments": "Primary NIC",
                "dependsOn": [
                    "[variables('publicIpAddressName')]",
                    "[variables('nsgName')]",
                    "[variables('virtualNetworkName')]"
                ],
                "properties": {
                    "ipConfigurations": [
                        {
                            "name": "ipconfig1",
                            "properties": {
                                "subnet": {
                                    "id": "[variables('subnetRef')]"
                                },
                                "privateIPAllocationMethod": "Dynamic",
                                "publicIpAddress": {
                                    "id": "[resourceId('Microsoft.Network/publicIpAddresses', variables('publicIpAddressName'))]"
                                }
                            }
                        }
                    ],
                    "networkSecurityGroup": {
                        "id": "[resourceId('Microsoft.Network/networkSecurityGroups', variables('nsgName'))]"
                    }
                }
            },
            {
                "name": "[variables('publicIpAddressName')]",
                "type": "Microsoft.Network/publicIpAddresses",
                "apiVersion": "[variables('networkApiVersion')]",
                "location": "[resourceGroup().location]",
                "comments": "Public IP for Primary NIC",
                "properties": {
                    "publicIpAllocationMethod": "Dynamic"
                }
            },
            {
                "name": "[variables('nsgName')]",
                "type": "Microsoft.Network/networkSecurityGroups",
                "apiVersion": "[variables('networkApiVersion')]",
                "location": "[resourceGroup().location]",
                "comments": "Network Security Group (NSG) for Primary NIC",
                "properties": {
                    "securityRules": [
                        {
                            "name": "default-allow-rdp",
                            "properties": {
                                "priority": 1000,
                                "sourceAddressPrefix": "*",
                                "protocol": "Tcp",
                                "destinationPortRange": "3389",
                                "access": "Allow",
                                "direction": "Inbound",
                                "sourcePortRange": "*",
                                "destinationAddressPrefix": "*"
                            }
                        }
                    ]
                }
            }
        ],
        "outputs": {}
    }
    ```

1.  ラボ　コンピューターの「ファイルエクスプローラー」ウィンドウで、**C:\\Labfiles** フォルダーに **az400m05-vm0.deployment.template.parameters.json** という名前の新しいファイルを作成し、次の内容で保存して閉じます。 

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "vmSize": {
                "value": "Standard_D2s_v3"
            },
            "adminUsername": {
                "value": "Student"
            },
            "adminPassword": {
                "value": "Pa55w.rd1234"
            }
        }
    }
    ```    

1.  「**管理者: Windows PowerShell ISE**」 ウィンドウのスクリプト ペインから、以下を実行して、このラボの最初に生成したカスタム イメージを使用してセルフホスティド エージェントをプロビジョニングするために使用される Azure VM をデプロイします。

    ```powershell
    Set-Location -Path 'C:\Labfiles'
    New-AzResourceGroupDeployment -ResourceGroupName $rgName -Name az400m05-vm0-deployment -TemplateFile .\az400m05-vm0.deployment.template.json -TemplateParameterFile .\az400m05-vm0.deployment.template.parameters.json -storageAccountName $storageAccountName -imageUrl $imageUrl
    ```

    > **注**: プロビジョニングプロセスが完了するまで待ちます。これにはおよそ 5 分かかります。 


#### タスク 2: Azure DevOps セルフホスティングエージェントを構成する

このタスクでは、新しくプロビジョニングされた Azure VM を Azure DevOps セルフホスティング エージェントとして構成し、それを使用してビルド パイプラインを実行します。

1.  ラボ コンピューターで、Azure portal を表示する Web ブラウザー ウィンドウに切り替え、Azure ポータルで、**仮想マシン**のリソースの種類を検索して選択し、**仮想マシン** ブレードで **az400m05-vm0** をクリックします。
1.  **az400m05-vm0** ブレードで、「**接続**」 をクリックし、ドロップダウン リストで 「**RDP**」 をクリックし、**az400m05-vm0 \| 接続**ブレードで 「**RDP ファイルのダウンロード**」 をクリックし、ダウンロードした RDP ファイルを開いて、リモート デスクトップを使用して **az400m05-vm0** Azure VM に接続します。サインインするように求められたら、ユーザー名として「**Student**」を入力し、パスワードとして「**Pa55w.rd1234**」を入力します。
1.  **az400m05-vm0** へのリモート デスクトップ セッション内で、Web ブラウザーを起動し、[Azure DevOps ポータル](https://dev.azure.com)に移動し、Azure DevOps 組織に関連付けられている Microsoft アカウントを使用してサインインします。
1.  Azure DevOps ポータルで、「**エージェントの取得**」 パネルを閉じ、「Azure DevOps」 ページの右上隅にある 「**ユーザー設定**」 アイコンをクリックし、ドロップダウン メニューで、「**パーソナル アクセス トークン**」 をクリックし、「**パーソナル アクセス トークン**」 ペインで 「**+ 新しいトークン**」 をクリックします。。
1.  「**新しいパーソナル アクセス トークンの作成**」 ペインで、「**すべてのスコープを表示**」 リンクをクリックし、次の設定を指定して 「**作成**」 をクリックします (他のすべてのスコープはデフォルト値のままにします)。

    | 設定 | 値 |
    | --- | --- |
    | 名前 | **エージェント プールの構成とパイプライン スタイルの理解ラボ** |
    | 適用範囲 | **エージェント プール** |
    | アクセス許可 | **読み取りと管理** |

1.  「**成功**」 ペインで、パーソナル アクセス トークンの値をクリップボードにコピーします。

    > **注**: 必ずトークンをコピーしてください。このペインを閉じると、取得できなくなります。 

1.  「**成功**」 ペインで、「**閉じる**」 をクリックします。
1.  Azure DevOps ポータルの 「**パーソナル アクセス トークン**」 ペインで、左上隅にある 「**Azure DevOps**」 記号をクリックしてから、左下隅にある 「**組織の設定**」 ラベルをクリックします。
1.  「**概要**」 ペインの左側の垂直メニューの 「**パイプライン**」 セクションで、「**エージェント プール**」 をクリックします。
1.  「**エージェント プール**」 ペインの右上隅にある 「**プールの追加**」 をクリックします。 
1.  「**エージェント プールの追加**」 ペインの 「**プールの種類**」 ドロップダウン リストで、「**セルフホスティド**」 を選択し、「**名前**」 テキスト ボックスに「**az400m05l05a-pool**」と入力して、「**作成**」 をクリックします。
1.  「**エージェント プール**」 ペインに戻り、新しく作成された **az400m05l05a-pool** を表すエントリをクリックします。 
1.  **az400m05l05a-pool** ペインの 「**ジョブ**」 タブで、「**エージェント**」 ヘッダーをクリックします。
1.  **az400m05l05a-pool** ペインの 「**エージェント**」 タブで、「**新しいエージェント**」 ボタンをクリックします。
1.  「**エージェントの取得**」 ペインで、「**Windows**」 タブと 「**x64**」 タブが選択されていることを確認し、「**ダウンロード**」 をクリックしてエージェント バイナリを含む zip アーカイブをダウンロードし、ユーザー プロファイル内のローカルの**ダウンロード** フォルダーにダウンロードします。

    > **注**: この時点で、現在のシステム設定でファイルのダウンロードができないことを示すエラーメッセージが表示された場合は、「インターネット エクスプローラー」 ウィンドウの右上隅にある、ドロップダウン メニューの 「**設定**」 メニュー ヘッダーを示す歯車の記号をクリックします。「**インターネット オプション**」 を選択し、「**インターネット オプション**」 ダイアログ ボックスで 「**詳細設定**」 をクリックし、「**詳細設定**」 タブで 「**リセット**」 をクリックし、「**インターネット エクスプローラー設定のリセット**」 ダイアログ ボックスで 「**リセット**」 をクリックし、「**閉じる**」 をクリックして、ダウンロードを再試行します。 

1.  **az400m05-vm0** へのリモート デスクトップ セッション内で、管理者として Windows PowerShell を起動し、「**管理者: Windows PowerShell**」 コンソールから、次を実行して、**C:\\agent** ディレクトリを作成し、ダウンロードしたアーカイブのコンテンツをそこに抽出します：

    ```powershell
    New-Item -Type Directory -Path 'C:\agent'
    Set-Location -Path 'C:\agent'
    Add-Type -AssemblyName System.IO.Compression.FileSystem
    [System.IO.Compression.ZipFile]::ExtractToDirectory("$HOME\Downloads\vsts-agent-win-x64-2.179.0.zip", "$PWD")
    ```

1.  **az400m05-vm0** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell**」 コンソールで、以下を実行して、エージェントを構成します。

    ```powershell
    .\config.cmd
    ```

1.  プロンプトが表示されたら、次の設定の値を指定します。

    | 設定 | 値 |
    | ------- | ----- |
    | サーバーの URL を入力する | **https://dev.azure.com/`<organization_name>`** の形式でのAzure DevOps 組織の URL。ここで、`<organization_name>` は、Azure DevOps 組織の名前を表します |
    | 認証タイプを入力する (PAT の場合は Enter 押します) | **Enter** キー | 
    | パーソナル アクセス トークンを入力する | このタスクの前半で記録したアクセス トークン |
    | エージェント・プールを入力する (デフォルトで Enter を押します) | **az400m05l05a-pool** |
    | エージェント名を入力する | **az400m05が-VM0** |
    | 作業フォルダーを入力する (_work の場合は Enter 押します) | **C:\agent** |
    | 「各ステップのタスクの解凍を実行する」と入力します。(N の場合は Enter を押します) | **Enter** |
    | 「エージェントをサービスとして実行しますか?」入力します。(Y/N) (N の場合は Enter を押します) | **Enter** |
    | 「自動ログオンを設定し、起動時にエージェントを実行する」と入力します (Y/N) (N の場合は Enter を押します) | **Enter** |

    > **注**: セルフホスティド エージェントは、サービスまたは対話型プロセスとして実行できます。エージェントの機能の検証が簡単になるため、対話型モードから始めることをお勧めします。本番環境で使用する場合は、エージェントをサービスとして実行するか、自動ログオンを有効にした対話型プロセスとして実行することを検討する必要があります。どちらも実行状態を維持し、オペレーティング システムが再起動された場合にエージェントが自動的に起動するようにするためです。

1.  **az400m05-vm0** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell**」 コンソールから、以下を実行して、エージェントを対話型モードで起動します。

    ```powershell
    .\run.cmd
    ```

    > **注**: エージェントが「**ジョブのリッスン**」ステータスを報告していることを確認します。

1.  Azure DevOps ポータルを表示しているブラウザー ウィンドウに切り替えて、「**エージェントの取得**」 ペインを閉じます。
1.  **az400m05l05a-pool** ペインの 「**エージェント**」 タブに戻り、新しく構成されたエージェントが 「**オンライン**」 ステータスで一覧表示されていることに注意してください。
1.  Azure DevOps ポータルを表示している Web ブラウザー ウィンドウの左上隅にある、**Azure DevOps** ラベルをクリックします。
1.  プロジェクトのリストを表示しているブラウザー ウィンドウで、**エージェント プールの構成とパイプライン スタイルの理解**プロジェクトを表すタイルをクリックします。 
1.  「**エージェント プールの構成とパイプライン スタイルの理解**」 ペインの左側の垂直ナビゲーション ペインの 「**パイプライン**」 セクションで、「**パイプライン**」 をクリックします。 
1.  「**パイプライン**」 ペインの 「**最近**」 タブで 「**PartsUnlimited**」 を選択し、「**PartsUnlimited**」 ペインで 「**編集**」 を選択します。
1.  **PartsUnlimited** 編集ペインの既存の YAML ベースのパイプラインで、**6** 行目の `vmImage：vs2017-win2016` を置き換えて、ターゲット エージェント プールを指定し、次のコンテンツを指定し、新しく作成されたセルフホスティド エージェント プールを指定します：

    ```yaml
    name: az400m05l05a-pool
    demands:
    - agent.name -equals az400m05-vm0
    ```

1.  **PartsUnlimited** 編集ペインのペインの右上隅にある 「**保存**」 をクリックし、「**保存**」 ペインでもう一度 「**保存**」 をクリックします。これにより、このパイプラインに基づいてビルドが自動的にトリガーされます。 
1.  Azure DevOps ポータルの左側の垂直ナビゲーション ペインの 「**パイプライン**」 セクションで、「**パイプライン**」 をクリックします。
1.  「**パイプライン**」 ペインの 「**最近**」 タブで、「**PartsUnlimited**」 エントリをクリックし、「**PartsUnlimited**」 ペインの 「**実行**」 タブで最新の実行を選択し、実行の 「**概要**」 ペインで一番下までスクロールし、「**ジョブ**」 セクションで 「**フェーズ 1**」 をクリックし、正常に完了するまでジョブを監視します。 

### 演習 3: Azure ラボ リソースを削除する

この演習では、このラボでプロビジョニングされた Azure リソースを削除して、予期しない料金を排除します。 

> **注**: 新しく作成した Azure リソースのうち、使用しないリソースは必ず削除してください。使用しないリソースを削除しないと、予期しないコストが発生する場合があります。

#### タスク 1: Azure ラボ リソースを削除する

このタスクでは、Azure Cloud Shell を使用して、このラボでプロビジョニングされた Azure リソースを削除し、不要な料金を排除します。 

1.  Azure portal で、**Cloud Shell** ウィンドウ内で **Bash** シェル セッションを開きます。
1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを表示します。

    ```sh
    az group list --query "[?starts_with(name,'az400m05')].name" --output tsv
    ```

1.  次のコマンドを実行して、このモジュールのラボ全体で作成したすべてのリソース グループのリストを削除します。

    ```sh
    az group list --query "[?starts_with(name,'az400m05')].「name」" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    > **注**: コマンドは非同期に実行されるので (--nowait パラメーターで決定される)、同じ Bash セッション内ですぐに別の Azure CLI コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

#### レビュー

このラボでは、従来のパイプラインを YAML ベースのパイプラインに変換する方法と、セルフホスティド エージェントを実装して使用する方法を学習しました。
