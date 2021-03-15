---
lab:
  title: "ラボ: GitHub Actions継続的インテグレーション"
  module: "モジュール 8: GitHub Actionsとの継続的インテグレーションの実装"
---

# ラボ: GitHub Actions の継続的インテグレーション

# 学生用ラボ マニュアル

## ラボの概要

GitHub Actions は、継続的インテグレーション (CI) を GitHub リポジトリに簡単に組み込むことができます。このラボでは、ビルド プロセスを自動化する 2 つの GitHub ワークフローを設定します。

> **注**: このラボでは、**github-learning-lab** ボットを操作します。このボットは、個々のラボ タスクをガイドします。

## 目標

このラボを完了すると、次のことができるようになります。

- 継続的インテグレーションにおける GitHub Actions の重要性を説明する
- テンプレート化されたワークフローを使用およびカスタマイズする
- チームのニーズと行動に一致する CI ワークフローを作成する
- リポジトリのソース コードを使用して、ワークフロー内のジョブ全体で成果物を構築する
- GitHub Actions を使用してユニット テスト フレームワークを実装する
- テストを実行し、テスト レポートを作成するワークフローを作成する
- マトリックス ビルドを設定して、複数のターゲット プラットフォームのビルド成果物を作成する
- リポジトリのビルド成果物を保存してアクセスする
- アプリケーションの CI ニーズに合わせて仮想環境を選択する

> **注**: このラボでは、タスクの多くがビルドに関連します。これらのビルドは、ビルドに数分かかる場合があります。次のステップに進む前に、ビルドが完了するのを必ず待ってください。

## ラボの所要時間

- 推定時間: **150 分**

## 手順

### 開始する前に

#### ラボの仮想マシンにログインする

次の資格情報を使用して、Windows 10 コンピューターにサインインしていることを確認します。

- ユーザー名: **Student**
- パスワード: **Pa55w.rd**

#### このラボに必要なアプリケーションを確認する

このラボで使用するアプリケーションを特定する:

- Microsoft Edge

#### GitHub アカウントを設定する

このラボで使用できる GitHub アカウントをまだお持ちでない場合は、[新しい GitHub アカウントのサインアップ](https://docs.github.com/en/free-pro-team@latest/github/getting-started-with-github/signing-up-for-a-new-github-account)にある手順に従ってください。

### 演習 1: GitHub Actions との継続的インテグレーションを実装する

この演習では、GitHub Actions との継続的インテグレーションを実装します

#### タスク 1: テンプレート化されたワークフローを GitHub リポジトリに追加する

このタスクでは、次の一連の手順を使用して、テンプレート化されたワークフローを使用して Pull request を作成します。

- 「Actions」 タブに移動します。
- テンプレート Node.js ワークフローを選択します。
- ワークフローを新しいブランチにコミットします。
- Node の CI というタイトルの Pull request を作成します。

1.  ラボ コンピューターで、Web ブラウザーを起動し、[GitHub Actions: Continuous Integration](https://lab.github.com/githubtraining/github-actions:-continuous-integration)の起動ページに移動します。GitHub にまだサインインしていない場合は、右上隅にある 「**Sign in**」 をクリックします。
1.  「**GitHub Learning Lab**」 ページで、「**Start learning with GitHub Learning Lab**」 をクリックします。
1.  [GitHub Actions: Continuous Integration ラボ](https://lab.github.com/githubtraining/github-actions:-continuous-integration),の起動ページに戻り、「**Start free course**」 をクリックします。
1.  ポップアップウィンドウが開くので、「**Begin GitHub Actions: Continuous Integration**」 をクリックします。**GitHub Learning Lab** が GitHub アカウントに **github-actions-for-ci** という名前のパブリック リポジトリを作成します。
1.  「**GitHub Actions: Continuous Integration**」 ページに戻り、「**Course steps**」 のリストで、最初のステップ 「**Use a templated workflow**」 の横にある 「**Start**」 をクリックします。**github-actions-for-ci** リポジトリの 「**Issues**」 タブに自動的にリダイレクトされます。
1.  **github-actions-for-ci** リポジトリの 「**Issues**」 タブの 「**There's a bug**」 の問題ページで、「**Welcome**」 セクションを確認します。

    > **注**: 「**Welcome**」 セクションによると、リポジトリのどこかにバグがあります。継続的インテグレーション (CI) の実践を使用して自動テストを設定し、このようなシナリオの発見、診断、および最小化を容易にします。コードベースは Node.js で書かれています。GitHub Actions を使用すると、Node.js などの一般的な言語やフレームワーク用にテンプレート化されたワークフローを使用できます。テンプレート化されたワークフローを使用して Pull request を作成します。

1.  「**There's a bug**」 ページで、「**Actions**」 トップ メニューのタブ ヘッダーをクリックします。
1.  **github-actions-for-ci** リポジトリの 「**Actions**」 タブの 「**Get started with GitHub Actions**」 ページの 「**Node.js**」 ペインで、「**Set up this workflow**」 をクリックします。
1.  **github-actions-for-ci/.github/workflows/node.js.yml** ページで、「**Start commit**」 をクリックします。
1.  「**Commit new file**」 ペインで、このコミットの新しいブランチを作成するデフォルト設定を受け入れて Pull request を開始し、「**Commit new file**」 をクリックします。
1.  「**Open a pull request**」 ページで、Pull request の名前を、「**Create node.js.yml**」 から「**CI for Node**」 に変更し、「**Create pull request**」 をクリックします。

> **注**: テンプレート化されたワークフローをブランチに追加するだけで、GitHub Actions がリポジトリの CI を開始できます。

#### タスク 2: テンプレート化されたワークフローを実行する

このタスクでは、テンプレート化されたワークフローの実行を追跡し、結果を確認します

1.  **CI for Node #2** の 「**Pull requests**」 タブの「**Conversation**」 で、意図した変更を確認します。これは、**github/workflows/node.js.yml** ファイルの内容を表します。

1.  個々の変更を確認するときに、対応する 「**Resolve conversation**」 ボタンをクリックします。

    > **注**: **github/workflows/node.js.yml** ファイルには次のコンポーネントが含まれています。

    - Workflow: Workflow は自動化の単位であり、自動化をトリガーするもの、自動化中に考慮すべき環境またはその他の側面、およびトリガーの結果として発生する必要があるものの定義が含まれます。
    - Job: Job は、1 つ以上のステップで構成されるワークフローのセクションです。サンプル ワークフローでは、テンプレートがビルド ジョブを形成するステップを定義します。
    - Step: Step は、自動化の一部を表します。ステップは GitHub Actions として定義できます。
    - Action: GitHub Actions は、ワークフローと互換性のある方法で記述された自動化の一部です。GitHub から直接入手できる、オープンソース コミュニティから入手できる組み込み Actions を使用することも、カスタム Actions を作成することもできます。たとえば、**actions/checkout@v2** は、ビルドを実行している仮想マシンにコードベースのコピーがあることを確認するために使用されます。チェックアウトされたコードは、テストの実行に使用されます。複数のバージョンに対してテストを実行するため、Actions **actions/setup-node@v1** を使用して Node.js の適切なバージョンをセットアップします。

    > **注**: **github/workflows/node.js.yml** ファイルの **on:** フィールドは、GitHub Actions にいつ実行するかを手順します。この場合、プッシュがあるときはいつでもワークフローを実行しています。

    > **注**: **github/workflows/node.js.yml** ファイルの **jobs:** ブロックは、Actions ワークフローのコア コンポーネントを定義します。ワークフローはジョブで構成されており、テンプレート ワークフローは識別子 build を使用して単一のジョブを定義します。すべてのジョブには、実行する特定のホストマシンも必要です。これは、**runs-on:** フィールドの値で指定されます。このテンプレート ワークフローは、最新バージョンの Ubuntu を使用してビルド ジョブを実行しています。

    > **注**: ワークフローは、ビルド済みの Actions の実行に加えて、ビルドを実行している仮想マシンに直接アクセスする場合と同じように、コマンドを実行することもできます。テンプレート ワークフローの **run:** フィールドを使用すると、依存関係をインストールする **npm install** や、選択したテスト フレームワークを実行する **npm test** など、プロジェクトに関連する任意のコマンドを実行できます。

    ```yaml
    # このワークフローは、Nodeの依存関係をクリーン インストールし、ソース コードをビルドし、Nodeのさまざまなバージョンでテストを実行します
    # 詳細については、https://help.github.com/actions/language-and-framework-guides/using-nodejs-with-github-actions を参照してください。

    name: Node.js CI

    on:
      push:
        branches: [main]
      pull_request:
        branches: [main]

    jobs:
      build:
        runs-on: ubuntu-latest

        strategy:
          matrix:
            node-version: [10.x, 12.x, 14.x, 15.x]

        steps:
          - uses: actions/checkout@v2
          - name: Use Node.js ${{ matrix.node-version }}
            uses: actions/setup-node@v1
            with:
              node-version: ${{ matrix.node-version }}
          - run: npm ci
          - run: npm run build --if-present
          - run: npm test
    ```

> **注**: ワークフローが失敗するという事実は無視してください。これは予測されていることです。失敗の理由を特定するには、リポジトリの 「**Actions**」 タブの内容を確認するか、Pull request の 「**Conversation**」 タブにリストされている失敗に対応する 「**Details**」 リンクをクリックします。

> **注**: この場合、エラーの原因は **npm test** コマンドです。**npm test** コマンドは、テスト フレームワークを探します。このラボでは、Jest を使用します。Jest では、**\_\_test\_\_**.という名前のディレクトリで単体テストを行う必要があります。問題は、**\_\_test\_\_** ディレクトリがこのブランチに存在しないことです。

#### タスク 3: GitHub ワークフローに Jest テストを追加する

このタスクでは、Jest ベースのテストを GitHub ワークフローに追加して、次の手順を使用して、前のタスクで特定した失敗に対処します。

- **Add Jest tests** という名前の、オープンな Pull request に移動
- Pull request をマージ

1. 「**Pull request**」 タブをクリックします。
1. Pull request のリストで、「**Add Jest tests**」 をクリックします。
1. 「**Add Jest tests**」 ページで、Pull request の 「**Conversation**」 タブの内容を確認します。

   > **注**: この Pull request では、良く使用される JavaScript テスト フレームワークである Jest が導入されています。これを使用して、継続的インテグレーションに使用する方法を学習します。

1. 「**Add Jest tests**」 ページの Pull request の 「**Conversation**」 タブで、「**Merge pull request**」 をクリックしてから、「**Confirm merge**」 をクリックします。
1. 「**Add Jest tests**」 ページの「**Conversation**」 タブで、**github-learning-lab** ボットからのコメントで 「**next step**」 リンクをクリックします。これにより、**github-actions-for-ci** GitHub リポジトリの**CI for Node #2** Pull request の 「**Conversation**」 タブにリダイレクトされます。

#### タスク 4: Actions ログを確認して失敗したテストを特定する

このタスクでは、Actions ログを確認して、失敗したテストを特定します。

> **注**: テスト フレームワークが適切に構成されたので、ビルド プロセスが自動的にそれを呼び出す必要があります。対応するログを確認して結果を判断します。次のステップを推奨します。以前と同様に、リポジトリの 「**Actions**」 タブ、または Pull request の 「**Conversation**」 タブにリストされている失敗に対応する 「**Details**」 リンクからログにアクセスできます。

このタスクを完了するには、次の一連の手順を使用します。

- ワークフロー ログに移動する
- 失敗したテストの名前を特定する
- 失敗したテストの名前を含むコメントを Conversation に追加する

1. **CI for Node #2** の 「**Conversation**」 タブに戻り、**github-learning-lab** ボットから 「**Waiting on tests**」 コメントまでスクロールして、その内容を確認します。
1. 「**Conversation**」 タブの下にスクロールし、失敗した個々のチェックの横にある各 「**Details**」リ ンクをクリックします。これにより、**CI for Node #2** の 「**Checks**」 タブに自動的にリダイレクトされます。
1. **CI for Node #2** の 「**Checks**」 タブで、各ビルド ログを確認し、**Run npm test**中にすべての失敗があることを確認します。
1. **Run npm test**ステージを調べ、「**Game**」 セクションで、失敗を示す **X** マークが付いている個々のテストの名前を特定します。

   ```
   FAIL __test__/game.test.js
     App
       ✕ Contains the compiled JavaScript (8 ms)
     Game
       Game
         ✕ Initializes with two players (3 ms)
         ✓ Initializes with an empty board (1 ms)
         ✕ Starts the game with a random player
       turn
         ✓ Inserts an 'X' into the top center
         ✓ Inserts an 'X' into the top left
       nextPlayer
         ✕ Sets the current player to be whoever it is not (1 ms)
       hasWinner
         ✓ Wins if any row is filled (1 ms)
         ✓ Wins if any column is filled
         ✓ Wins if down-left diagonal is filled (1 ms)
         ✓ Wins if up-right diagonal is filled
   ```

1. 「**Conversation**」 タブに戻り、コメントのリストの一番下までスクロールし、最後のコメントの 「**Write**」 タブで、「**Leave a comment**」 を前の手順で特定したテストの名前に置き換えて、「**Comment**」 をクリックします。

   ```
   Initializes with two players
   Starts the game with a random player
   Sets the current player to be whoever it is not
   ```

> **注**: これにより、「**Reading failed logs**」から始まる、ボットからの別のコメント セットが 「**Conversation**」 タブに自動的に表示されます。

#### タスク 5: 失敗したテストを修正する

このタスクでは、問題を修正するために、テストが失敗する原因となっているファイルを変更します。

> **注**: 失敗したテストの 1 つは次のとおりです。2 人のプレイヤーで初期化します。ログをさらに詳しく調べてみると、単体テストでは 2 人のプレイヤーの名前が Salem と Nate であることが期待されていることに気付くかもしれませんが、どういうわけか Nate の代わりに Bananas が使用されました：

    ```
    ● Game › Game › Starts the game with a random player

      expect(received).toBe(expected) // Object.is equality

      Expected: "Nate"
      Received: "Bananas"

        39 |
        40 |       Math.random = () => 0.6
      > 41 |       expect(new Game(p1, p2).player).toBe('Nate')
            |                                       ^
        42 |     })
        43 |   })
        44 |
    ```

> **注**: テスト ファイルにテストするコードファイルと同じ名前を付け、.test.js 拡張子を追加するのが一般的な方法です。game.test.js のテスト結果は、game.js の問題が原因であると推測できます。

1. **CI for Node #2** Pull request の 「**Conversation**」 タブで、**github-learning-lab** ボットからの最新のコメントを見つけてその内容を確認します。
1. コメントで参照されているコードの変更を特定するには、「**View changes**」 ボタンをクリックし、**\_\_test\_\_/game.test.js** ファイルを表すセクションまでスクロールして、提案された変更を確認します。

   ```yaml
   Suggested change
   this.p2 = 'Bananas'
   this.p2 = p2
   ```

1. **CI for Node #2** Pull request の 「**Conversation**」 タブに戻り、**github-learning-lab** ボットから最新のコメントまでスクロールダウンし、「**Commit suggestion**」 をクリックして、ポップアップ ペインで 「**Commit changes**」 をクリックします。

   > **注**: 変更をコミットすると、テストが再度実行され、今回は正常に完了します。

1. **CI for Node #2** Pull request の 「**Conversation**」 タブで、「**Changes approved**」 応答まで下にスクロールし、「**Merge pull request**」 をクリックしてから、「**Confirm merge**」 をクリックします。
1. **CI for Node #2** Pull request の 「**Conversation**」 タブで、**github-learning-lab** ボットからの最新のコメントで 「**next step**」 リンクをクリックします。これにより、**A workflow for the entire team #4**の 「**Issues**」 タブにリダイレクトされます。

#### タスク 6: 次のステップを確認する

このタスクでは、最初のワークフローを共有してチーム全体で使用できるようにする次のステップを確認します。

> **注**: CI の設定方法を学習したので、より現実的なユースケースを試してみましょう。あなたのチームには、これまで使用してきたテンプレートを超えるカスタム ワークフローがあります。次の機能が必要です。

- オペレーティング システムと Node.js バージョンのさまざまな組み合わせを検証できるように複数のターゲットに対してテストする機能
- ビルドをテストの詳細から分離できるようにするための専用のテスト ジョブ
- ターゲット環境にアーティファクトをデプロイし、アーティファクトをビルドするためにアクセスすること。
- マスター ブランチが削除されたり、誤って破損したりしないようにするためのブランチ保護
- チームメートが Pull request を再確認できるようにするために必要なレビュー
- 迅速にマージし、マージとデプロイを自動化できる明確な承認

#### タスク 7: カスタム GitHub Actions ワークフローを作成する

このタスクでは、カスタム GitHub Actions ワークフローを作成します。

この機能を実装するには、次の一連の手順を実行します。

- 新しいブランチで existing workflow ファイルを編集する
- ワークフロー ファイルで、Node のバージョン 12.x および 14.x をターゲットにする
- 変更を含む **Improve CI**という名前の新しい Open a pull request

1.  「**Issues**」 タブの**A workflow for the entire team #4** ページで、「**Step 7: Create a custom GitHub Actions workflow**」というラベルの付いたセクションまでスクロールします。「**Activity: Edit the existing workflow with new build targets**」 の配下にある、**existing workflow**リンクをクリックします。これにより、「**Code**」 タブにある **github-actions-for-ci/.github/Workflows/node.js.yml** ファイルの 「**Edit file**」 ビューにリダイレクトされます。
1.  **github-actions-for-ci/.github/workflows/node.js.yml** ファイルで、`node-version: [10.x, 12.x, 14.x, 15.x]` を `node-version: [12.x, 14.x]` に置き換えます。
1.  「**Code**」 タブの右上隅にある 「**Start commit**」 をクリックします。
1.  「**Commit changes**」 ペインで、このコミットの新しいブランチを作成するデフォルト設定を受け入れて Pull request を開始し、「**Commit changes**」 をクリックします。
1.  「**Open a pull request**」 ページで、「**Update node.js.yml**」 の名前を 「**Improve CI**」 に変更し、「**Create pull request**」 をクリックします。これにより、**github-actions-for-ci** リポジトリの 「**Pull request**」 タブの 「**Create CI #5**」 ページに自動的にリダイレクトされます。

> **注**: Node の特定のバージョンを対象とすることで、複数のオペレーティング システム、プラットフォーム、および言語バージョンにわたるテストを可能にするビルド マトリックスを構成しました。このトピックの詳細については、[GitHub Docs](https://help.github.com/en/articles/configuring-a-workflow#configuring-a-build-matrix) を参照してください。

#### タスク 8: Windows 環境を対象にする

このタスクでは、ワーク フローファイルを編集して Windows 環境用にビルドします

> **注**: アプリの Windows 環境へのデプロイをサポートしたいので、マトリックス ビルド構成に Windows を追加しましょう。

この機能を実装するには、次の一連の手順を実行します。

- **os** フィールドをワークフローの **strategy.matrix** セクションに追加する
- **ubuntu-latest** と **windows-2016** をターゲット オペレーティング システムのリストに追加する
- ワークフローの変更を既存のブランチにコミットする

  ```yaml
  node-version: [10.x, 14.x]
  os: [ubuntu-latest, windows-2016]
  node-version: [12.x, 14.x]
  ```

1.  **github-actions-for-ci** リポジトリの 「**Pull request**」 タブの 「**Improve CI #5**」 ページで、**github-learning-lab** ボットから最新のコメントまでスクロールし、その内容を確認します。
1.  「**Activity: Edit your workflow file to build for Windows environments**」 セクションで、ステップ 1 の**github/workflows/nodejs.yml** リンクをクリックします。 これにより、リポジトリ **github-actions-for-ci** の「**Code**」 タブにある **github-actions-for-ci/.github/Workflows/node.js.yml** ファイルの 「**Edit file**」 ビューにリダイレクトされます。
1.  **github-actions-for-ci/.github/workflows/node.js.yml** ファイルで、matrix セクションを確認します。
1.  「**Pull request**」 タブの 「**Improve CI #5**」 ページの、「**Activity: Edit your workflow file to build for Windows environments**」 セクションに戻り、「**Commit suggestion**」 をクリックし、ポップ アップ ペインで「**Commit changes**」 をクリックします。
1.  これにより、 「**Improve CI #5**」 ページのコンテンツが、**New Job**というラベルの付いた **github-learning-lab** ボットからの最新のコメントで更新されます。

> **注**: ログを確認すると、この時点で 4 つのビルドがあることがわかります。これは、2 つのオペレーティング システムのそれぞれについて、2 つのバージョンに対してテストを実行しているためです。

#### タスク 9: 複数のジョブを構成する

このタスクでは、ワークフロー ファイルを編集して、ビルド ジョブとテスト ジョブを分離します。

> **注**: 次に、専用のテスト ジョブを作成しましょう。これにより、ワークフローのビルド機能とテスト機能を、ワークフローがトリガーされたときに実行される複数のジョブに分割できます。

1.  **github-actions-for-ci** リポジトリの 「**Pull request**」 タブの 「**Improve CI #5**」 ページで、「**New Job**」 というラベルの付いた **github-learning-lab** ボットからの最新のコメントで、「**Step 9: Use multiple jobs**」 セクションで、ステップ 1 の**your workflow file**のリンクをクリックします。 これにより、ブラウザーは、リポジトリ **github-actions-for-ci** の「**Code**」 タブにある **github-actions-for-ci/.github/Workflows/node.js.yml** ファイルの 「**Edit file**」 ビューにリダイレクトされます。

1.  次のようにファイルを上書きし、ワークフローを、ビルド用のジョブと、テスト用のジョブの 2 つに分離します。

    ```yaml
    name: Node CI

    on: [push]

    jobs:
      build:
        runs-on: ubuntu-latest

        steps:
          - uses: actions/checkout@v2
          - name: npm install and build webpack
            run: |
              npm install
              npm run build

      test:
        runs-on: ubuntu-latest

        strategy:
          matrix:
            os: [ubuntu-latest, windows-2016]
            node-version: [12.x, 14.x]

        steps:
          - uses: actions/checkout@v2
          - name: Use Node.js ${{ matrix.node-version }}
            uses: actions/setup-node@v1
            with:
              node-version: ${{ matrix.node-version }}
          - name: npm install, and test
            run: |
              npm install
              npm test
            env:
              CI: true
    ```

1.  編集中の **github-actions-for-ci/.github/workflows/node.js.yml** ファイルのコンテンツを表示する 「**Code**」 タブで、右上隅にある 「**Start commit**」 をクリックします。
1.  「**Commit changes**」 ペインで、現在のブランチに直接変更をコミットするデフォルト設定を受け入れ、「**Commit changes**」 をクリックします。

> **注**: このコミットに続いて、ワークフローが再度実行されます。

#### タスク 10: 複数のジョブを実行する

このタスクでは、ワークフロー内の複数のジョブの結果を待つだけです。

> **注**: このタスクでは、Actions は必要ありません。

#### タスク 11: ジョブのビルド アーティファクトをアップロードする

このタスクでは、ワークフロー ファイルのアップロード Actions を使用して、ジョブのビルド アーティファクトを保存します。

> **注**: ビルドは成功したが、各テストジョブは失敗したことに気付く場合があります。これは、各ジョブが仮想環境の新しいインスタンスで実行されるため、ビルドで作成されたビルド アーティファクトをテスト ジョブで使用できないためです。これは、仮想環境の設計に固有の部分です。これに対処するために、組み込みのアーティファクト ストレージを使用して、あるジョブから作成されたアーティファクトを保存し、同じワークフロー内の別のジョブで使用できます。アーティファクトを使用すると、ジョブの完了後にデータを永続化し、同じワークフロー内の別のジョブとそのデータを共有できます。アーティファクトは、ワークフローの実行中に生成されたファイルまたはファイルのコレクションです。

> **注**: アーティファクトをアーティファクト ストレージにアップロードするには、GitHub によってビルドされた Actions actions/upload-artifacts を使用できます。

1.  「**Pull request**」 タブの 「**Improve CI #5**」 ページで、「**Use the upload**」 というラベルの付いた **github-learning-lab** ボットからの最新のコメントを確認します。「**Step 11: Upload a job's build artifacts**」 セクションの、ステップ 1 で**your workflow file**のリンクをクリックします。これにより、ブラウザーは、リポジトリ **github-actions-for-ci** の「**Code**」 タブにある **github-actions-for-ci/.github/Workflows/node.js.yml** ファイルの 「**Edit file**」 ビューにリダイレクトされます。
1.  ビルド ジョブ セクションで、ビルド ジョブを更新します。 **upload-artifacts** Actions を含めるように、次のように更新します。

    ```yaml
    build:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v2
        - name: npm install and build webpack
          run: |
            npm install
            npm run build
        - uses: actions/upload-artifact@main
          with:
            name: webpack artifacts
            path: public/
    ```

1.  編集中の **github-actions-for-ci/.github/workflows/node.js.yml** ファイルの右上隅にある 「**Start commit**」 をクリックします。
1.  「**Commit changes**」 ペインで、現在のブランチに直接変更をコミットするデフォルト設定を受け入れ、「**Commit changes**」 をクリックします。

> **注**: このコミットに続いて、ワークフローが再度実行されます。

#### タスク 12: ジョブのビルド アーティファクトをダウンロードする

このタスクでは、ワークフロー ファイルのダウンロード Actions を使用して、前のジョブのビルド アーティファクトにアクセスします。

> **注**: ビルド アーティファクトはアーティファクト ストレージにアップロードされますが、ワークフローを追跡する場合は、テスト ジョブがまだ失敗していることに気付くでしょう。これには 2 つの主な理由があります。

- 順次実行するように明示的に構成されていない限り、ジョブは並行して実行されます。
- 各ジョブは独自の仮想環境で実行されるため、アーティファクトをストレージにプッシュしましたが、テスト ジョブはそれらを取得する必要があります。

> **注**: これを修正するには、ビルドが終了した後にのみテストを実行して、アーティファクトを使用できるようにします。さらに、アーティファクトを指定されたアーティファクト ストアにコピーし、アップロード Actions を活用するために、別の GitHub 組み込み Actions **actions/download-artifact** を使用して、これらのアーティファクトをダウンロードします。

1.  「**Improve CI #5**」 ページに戻り、「**Conversation**」 タブで、**github-learning-lab** ボットからの最新のコメントまでスクロールしてその内容を確認します。
1.  **github-learning-lab** ボットからの最新のコメントの「**Step 12: Download a job's build artifacts**」 セクションの、ステップ 1 で**your workflow file**のリンクをクリックします。これにより、ブラウザーは、リポジトリ **github-actions-for-ci** の「**Code**」 タブにある **github-actions-for-ci/.github/Workflows/node.js.yml** ファイルの 「**Edit file**」 ビューにリダイレクトされます。
1.  テスト ジョブ セクションで、**needs: build** コンポーネントを追加して、ビルド ジョブが完了した後にのみ実行されるようにテスト ジョブを更新します。

    ```yaml
    test:
      needs: build
      runs-on: ubuntu-latest
    ```

1.  さらにテスト ジョブを更新して、次のコンテンツと一致するように、**download-artifacts** Actions を含めます。

    ```yaml
    test:
      needs: build
      runs-on: ubuntu-latest

      strategy:
        matrix:
          os: [ubuntu-latest, windows-2016]
          node-version: [12.x, 14.x]

      steps:
        - uses: actions/checkout@v2
        - uses: actions/download-artifact@master
          with:
            name: webpack artifacts
            path: public
        - name: Use Node.js ${{ matrix.node-version }}
          uses: actions/setup-node@v1
          with:
            node-version: ${{ matrix.node-version }}
        - name: npm install, and test
          run: |
            npm install
            npm test
          env:
            CI: true
    ```

1.  編集中の **github-actions-for-ci/.github/workflows/node.js.yml** ファイルのコンテンツを表示するリポジトリ **github-actions-for-ci** の 「**Code**」 タブで、右上隅にある 「**Start commit**」 をクリックします。
1.  「**Commit changes**」 ペインで、現在のブランチに直接変更をコミットするデフォルト設定を受け入れ、「**Commit changes**」 をクリックします。

> **注**: このコミットに続いて、ワークフローが再度実行されます。

> **注**: カスタム ワークフローは、次の機能を提供するようになりました。

- サポートされているオペレーティング システムと Node.js バージョンが機能しているかどうかを確認するために、複数のターゲットに対してテストすること
- ビルドをテストの詳細から分離できるようにするための専用のテスト ジョブ
- ターゲット環境にアーティファクトをデプロイし、アーティファクトをビルドするためにアクセスすること。

#### タスク 13: 改善された CI ワークフローをチームと共有する

このタスクでは、Pull request と改善されたワークフローをマスター ブランチにマージします。

1. 「**Pull request**」 タブの 「**Improve CI #5**」 ページに戻り、「**Conversation**」 タブで、**github-learning-lab** ボットからの最新のコメントまでスクロールしてその内容を確認します。
1. 「**Merge the CI**」 というラベルの付いた **github-learning-lab** ボットからの最新のコメントで、「**Step 13: Share the improved CI workflow with the team**」 セクションを確認します。

   > **注**: これまでに実装した要件のリストのステータスは次のとおりです。

   - サポートされているオペレーティング システムと Node.js バージョンが機能しているかどうかを確認するために、複数のターゲットに対してテストすること
   - ビルドをテストの詳細から分離できるようにするための専用のテスト ジョブ
   - ターゲット環境にアーティファクトをデプロイし、アーティファクトをビルドするためにアクセスすること。

   > **注**: 次のいくつかのステップでは、チームのワークフローに次の機能を提供する変更を加えます。

   - マスター ブランチが削除されたり、誤って破損したりしないようにするためのブランチ保護
   - チームメートが Pull request を再確認するために必要なレビュー
   - 迅速にマージし、マージとデプロイを自動化できる明確な承認

1. 「**Changes approved**」 セクションまでスクロールし、「**Merge pull request**」 をクリックしてから、「**Confirm merge**」 をクリックします。

#### タスク 14: レビュー プロセスを自動化する

このタスクでは、チームのレビュー プロセスを自動化するために、新しいワークフロー ファイルを追加します。

1.  **github-actions-for-ci** リポジトリの 「**Pull request**」 タブの 「**Improve CI #5**」 ページで、**github-learning-lab** ボットからの最新のコメントで 「**next step**」 リンクをクリックします。これにより、**A custom workflow #6** という名前の Pull request の 「**Conversation**」 タブにリダイレクトされます。
1.  **A custom workflow #6** Pull request の 「**Conversation**」 タブで、下にスクロールして、「**Step 14: Automate the review process**」 というラベルの付いたセクションを確認します。

    > **注**: GitHub Actions は、さまざまなイベント トリガーに対して複数のワークフローを実行できます。Node.js ワークフローと連携する新しい承認ワークフローを作成しましょう。

1.  「**Activity: Add a new workflow file to automate the team's review process**」 というラベルの付いたセクションで、Actions のリストで、ステップ 1 の 「**new file**」 リンクをクリックします。これにより、**github-actions-for-ci** リポジトリの 「**Code**」 タブにリダイレクトされ、**github-actions-for-ci/.github/Workflows/approval-workflow.yml** ファイルが開いた状態のエディター ページが表示されます。
1.  エディター ペイン内で、`name: AZ-400 Team approval workflow` と入力し、ページの一番下までスクロールして、「**Commit new file**」 をクリックします。

#### タスク 15: Actions を使用して Pull request のレビューを自動化する

このタスクでは、新しいワークフローでコミュニティ Actions を使用します

> **注**: ワークフローは、以下を実行するように構成できます。

- GitHub からのイベント
- スケジュールされた時間
- GitHub の外部のイベントが発生したとき

> **注**: これまで、Node.js ワークフローにプッシュ イベントを使用してきました。これは、リポジトリへのコード変更に対して Actions を実行する場合に意味があります。レビュー ワークフローでは、人間によるレビューを行いたいと考えています。たとえば、承認済みの Pull request のラベル付け Actions を使用して、Pull request をマージするのに十分なレビューを取得したことを簡単に確認できるようにします。**pull_request_review** イベントでトリガーして、レビュー ワークフローを準備しましょう。

1.  Pull request **A custom workflow #6** の 「**Conversation**」 タブに戻り、**github-learning-lab** ボットからの最新のコメントまでスクロールし、その内容を確認します。
1.  「**Step 15: Use an action to automate pull request reviews**」 セクションを確認します。
1.  このステップでは、前のステップで作成したワークフロー ファイルに **pull_request_review** を追加するだけでよいことに注意してください。
1.  「**Step 15: Use an action to automate pull request reviews**」セクション内で、「**Commit suggestion**」 をクリックし、ポップアップ ペインで 「**Commit changes**」 をクリックします。

#### タスク 16: 新しいワークフローで承認ジョブを作成する

このタスクでは、新しいワークフロー ファイルで、コミュニティ Actions を使用する New Job を作成します。

1.  Pull request **A custom workflow #6** の 「**Conversation**」 タブで、**github-learning-lab** ボットからの最新のコメントまでスクロールし、その内容を確認します。
1.  「**Step 16: Create an approval job in your new workflow**」 セクションを確認します。
1.  このステップでは、前のタスクで変更した同じワークフローファイルに、**labelWhenApproved** という名前の New Job を次の形式で追加する必要があることに注意してください。

    ```yaml
    jobs:
      labelWhenApproved:
        runs-on: ubuntu-latest
    ```

1.  「**Step 16: Create an approval job in your new workflow**」セクション内で、「**Commit suggestion**」をクリックして、ポップアップ ペインで 「**Commit changes**」 をクリックします。

#### タスク 17: 承認の自動化

このタスクでは、コミュニティ Actions を使用して、レビュー承認プロセスの一部を自動化します。

> **注**: 前のタスクでは、**labelWhenApproved** 識別子を使用してジョブを作成しました。ここで、**pullreminders/label-when-approved-action** という名前のコミュニティ作成 Actions を使用します。この Actions は、事前設定された数の承認後に、Pull request にラベルを追加します。これらのラベルは、何かをマージする準備ができていることをチームに視覚的に示すために使用できます。また、他の Actions やツールを使用して、必要な数の承認を受け取ったときに Pull request を自動的にマージする方法としても使用できます。

1.  Pull request **A custom workflow #6** の 「**Conversation**」 タブで、**github-learning-lab** ボットからの最新のコメントまでスクロールし、その内容を確認します。
1.  「**Step 17: Automate approvals**」 セクションを確認します。

    > **注**: これを実装するには、次のガイドラインを使用します。

    - ワークフロー ファイルには、**steps:** ブロックが必要です
    - ステップ名は任意です
    - コミュニティ Actions を使用するには、**uses:** キーワードを含めます
    - **label-when-approved-action** には、次の環境変数を持つ **env:** という名前のブロックが必要です。
      - **APPROVALS** は、ラベルを適用するために必要な承認の数です。このラボでは、**1** に設定します。
      - Actions でラベルを作成してこのリポジトリに適用できるようにするには、**GITHUB_TOKEN** が必要です。
      - **ADD_LABEL** は、承認の数に達したときに追加する必要があるラベルの名前です。その名前は任意です。

1.  **github-actions-for-ci** リポジトリの 「**Code**」 タブに切り替え、ブランチのリストで、**team-workflow** エントリを選択し、フォルダーのリストで、**.github /workflows** をクリックし、「**approval-workflow.yml**」 をクリックします。 **github-actions-for-ci/.github/workflows/approval-workflow.yml** のコンテンツを表示し、コードペインの右側にある鉛筆アイコンをクリックして編集モードに入ります。
1.  次のコンテンツを **approval-workflow.yml** ファイルに追加します。

    ```yaml
    steps:
      - name: Label when approved
        uses: pullreminders/label-when-approved-action@master
        env:
          APPROVALS: "1"
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          ADD_LABEL: "approved"
    ```

1.  「**Start commit**」 をクリックします。
1.  「**Commit changes**」 ペインで、現在のブランチに直接変更をコミットするデフォルト設定を受け入れ、「**Commit changes**」 をクリックします。

#### タスク 18: ブランチ保護を使用する

このタスクでは、マスター ブランチを保護することにより、自動レビュー プロセスを完了します。

> **注**: 保護されたブランチにより、リポジトリの共同編集者がブランチに取り消せない変更を加えることができなくなります。保護されたブランチを有効にすると、必要なステータ スチェックや必要なレビューなど、他のオプションのチェックと要件を有効にすることもできます。

1.  Pull request **A custom workflow #6** の 「**Conversation**」 タブに戻り、**github-learning-lab** ボットからの最新のコメントまでスクロールし、その内容を確認します。
1.  「**Step 18: Use branch protections**」 セクションを確認します。
1.  ページの一番上までスクロールして戻り、トップ メニューで 「**Settings**」 ヘッダーをクリックし、左側の垂直メニューで 「**Branches**」 をクリックします。
    > ウィンドウが小さい場合、「Settings」は「...」に隠れています。
1.  「**Branch protection rules**」 セクションで、「**Add rule**」 をクリックします。
1.  **Branch protection rules** の **Branch name pattern** を **main** に設定し、**Protect matching branches** 設定リストで、「**Require pull request reviews before merging**」と「**Require status checks to pass before merging**」を有効にし、各ビルド ジョブとテスト ジョブのそれぞれの横にあるチェックボックスを選択し、ページの一番下までスクロールして、「**Create**」 をクリックします。
1.  **A custom workflow #6** の 「**Conversation** 」 タブに戻り、 ボットの最新のコメントまでスクロールし、「**Step 18: Use branch protections**」 セクションを確認し、「**Activity: Complete the automated review process by protecting the master branch**」サブセクションのステップ 8 で、「**approve the requested review**」 リンクをクリックします。これにより、ブラウザー セッションが**A custom workflow** Pull request の 「**Files changed**」 タブにリダイレクトされます。
1.  **A custom workflow** Pull request の 「**Files changed**」 タブで、「**Review changes**」 をクリックし、「**Approve**」 オプションを選択して、「**Submit review**」 をクリックします。これにより、ブラウザー セッションが **github-actions-for-ci** リポジトリの 「**Pull request**」 タブの 「**A custom workflow #6**」 というラベルの付いたページにリダイレクトされ、**github-learning-lab** ボットからの最新のコメントが表示され、ワークフローが正常に完了したことを確認します。

## レビュー

このラボでは、ビルド プロセスを自動化する GitHu b ワークフローを設定する方法を学びました。
