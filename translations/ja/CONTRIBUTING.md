<!--
CO_OP_TRANSLATOR_METADATA:
{
  "original_hash": "90d0d072cf26ccc1f271a580d3e45d70",
  "translation_date": "2025-07-16T14:37:50+00:00",
  "source_file": "CONTRIBUTING.md",
  "language_code": "ja"
}
-->
# Contributing

このプロジェクトでは、貢献や提案を歓迎しています。ほとんどの貢献には、あなたが貢献物の使用権を持ち、実際に当社にその権利を付与していることを宣言するContributor License Agreement（CLA）への同意が必要です。詳細は[https://cla.opensource.microsoft.com](https://cla.opensource.microsoft.com)をご覧ください。

プルリクエストを提出すると、CLAボットが自動的にCLAの提出が必要かどうかを判断し、プルリクエストに適切なステータスチェックやコメントを付けます。ボットの指示に従うだけで、当社のCLAを使用しているすべてのリポジトリで一度だけ対応すれば問題ありません。

## Code of Conduct

このプロジェクトは[Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/)を採用しています。詳細は[Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/)をご覧いただくか、追加の質問やコメントがあれば[opencode@microsoft.com](mailto:opencode@microsoft.com)までご連絡ください。

## Cautions for creating issues

一般的なサポートの質問についてはGitHubのIssueを開かないでください。GitHubのIssueリストは機能要望やバグ報告に使用してください。こうすることで、実際のコードの問題やバグをより簡単に追跡でき、一般的な議論とコードの問題を分けて管理できます。

## How to Contribute

### Pull Requests Guidelines

Phi-3 CookBookリポジトリにプルリクエスト（PR）を提出する際は、以下のガイドラインに従ってください。

- **リポジトリのフォーク**：変更を加える前に必ず自分のアカウントにリポジトリをフォークしてください。

- **プルリクエストの分割**：
  - 変更の種類ごとに別々のプルリクエストを提出してください。例えば、バグ修正とドキュメントの更新は別々のPRにしてください。
  - タイポ修正や軽微なドキュメントの更新は、適切な場合は1つのPRにまとめても構いません。

- **マージコンフリクトの対応**：プルリクエストにマージコンフリクトがある場合は、ローカルの`main`ブランチをメインリポジトリに合わせて更新してから変更を加えてください。

- **翻訳の提出**：翻訳のPRを提出する際は、翻訳フォルダに元のフォルダ内のすべてのファイルの翻訳が含まれていることを確認してください。

### Writing Guidelines

すべてのドキュメントで一貫性を保つため、以下のガイドラインを守ってください。

- **URLの書式**：すべてのURLは角括弧で囲み、その後に丸括弧でURLを続けてください。角括弧や丸括弧の内外に余分なスペースを入れないでください。例：[example](https://www.microsoft.com)。

- **相対リンク**：現在のディレクトリ内のファイルやフォルダへの相対リンクには`./`を、親ディレクトリ内のものには`../`を使ってください。例：[example](../../path/to/file)や[example](../../../path/to/file)。

- **国別ロケールを含めない**：リンクに国別ロケールを含めないようにしてください。例として`/en-us/`や`/en/`は避けてください。

- **画像の保存場所**：すべての画像は`./imgs`フォルダに保存してください。

- **画像の名前付け**：画像名は英数字とダッシュを使い、わかりやすい名前にしてください。例：`example-image.jpg`。

## GitHub Workflows

プルリクエストを提出すると、変更を検証するために以下のワークフローが実行されます。プルリクエストがワークフローチェックを通過するよう、以下の指示に従ってください。

- [Check Broken Relative Paths](../..)
- [Check URLs Don't Have Locale](../..)

### Check Broken Relative Paths

このワークフローは、ファイル内のすべての相対パスが正しいかを確認します。

1. リンクが正しく機能しているか確認するには、VS Codeを使って以下の操作を行ってください：
    - ファイル内の任意のリンクにカーソルを合わせる。
    - **Ctrl + クリック**でリンク先に移動する。
    - ローカルでリンクが機能しない場合、そのリンクはGitHub上でも機能せず、ワークフローが失敗します。

1. 問題を修正するには、VS Codeのパス候補を使って以下の操作を行ってください：
    - `./`または`../`と入力する。
    - VS Codeが入力に基づいて利用可能な候補を表示します。
    - 目的のファイルやフォルダをクリックして正しいパスを選択してください。

正しい相対パスを追加したら、保存して変更をプッシュしてください。

### Check URLs Don't Have Locale

このワークフローは、ウェブURLに国別ロケールが含まれていないかを確認します。このリポジトリはグローバルにアクセス可能なため、URLに国別ロケールが含まれていないことが重要です。

1. URLに国別ロケールが含まれていないか確認するには、以下をチェックしてください：

    - URL内に`/en-us/`、`/en/`、またはその他の言語ロケールが含まれていないか。
    - これらが含まれていなければ、このチェックは合格です。

1. 問題を修正するには、以下の操作を行ってください：
    - ワークフローで指摘されたファイルを開く。
    - URLから国別ロケールを削除する。

国別ロケールを削除したら、保存して変更をプッシュしてください。

### Check Broken Urls

このワークフローは、ファイル内のウェブURLが正常に動作し、200ステータスコードを返しているかを確認します。

1. URLが正しく動作しているか確認するには、以下を行ってください：
    - ファイル内のURLのステータスをチェックする。

2. 壊れたURLを修正するには、以下を行ってください：
    - 壊れたURLを含むファイルを開く。
    - URLを正しいものに更新する。

URLを修正したら、保存して変更をプッシュしてください。

> [!NOTE]
>
> URLチェックが失敗する場合でも、リンクがアクセス可能なことがあります。これは以下のような理由によることがあります：
>
> - **ネットワーク制限**：GitHub Actionsのサーバーが特定のURLへのアクセスを制限している場合。
> - **タイムアウト問題**：応答に時間がかかりすぎるURLがタイムアウトエラーを引き起こす場合。
> - **一時的なサーバー問題**：サーバーの一時的なダウンやメンテナンスにより、検証時にURLが一時的に利用できない場合。

**免責事項**：  
本書類はAI翻訳サービス「[Co-op Translator](https://github.com/Azure/co-op-translator)」を使用して翻訳されました。正確性の向上に努めておりますが、自動翻訳には誤りや不正確な部分が含まれる可能性があります。原文の言語によるオリジナル文書が正式な情報源とみなされるべきです。重要な情報については、専門の人間による翻訳を推奨します。本翻訳の利用により生じたいかなる誤解や誤訳についても、当方は一切の責任を負いかねます。