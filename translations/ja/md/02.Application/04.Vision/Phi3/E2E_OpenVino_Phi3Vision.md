<!--
CO_OP_TRANSLATOR_METADATA:
{
  "original_hash": "69d48385b1f1b31dd20dbb2405031bff",
  "translation_date": "2025-04-04T13:02:14+00:00",
  "source_file": "md\\02.Application\\04.Vision\\Phi3\\E2E_OpenVino_Phi3Vision.md",
  "language_code": "ja"
}
-->
このデモでは、事前学習済みモデルを使用して、画像とテキストプロンプトに基づいてPythonコードを生成する方法を紹介します。

[サンプルコード](../../../../../../code/06.E2E/E2E_OpenVino_Phi3-vision.ipynb)

以下はステップごとの説明です：

1. **インポートとセットアップ**:
   - 必要なライブラリやモジュールをインポートします。画像処理用の`requests`、`PIL`、モデル処理用の`transformers`などが含まれます。

2. **画像の読み込みと表示**:
   - `demo.png`で指定された画像ファイルを`PIL`ライブラリを使用して開き、表示します。

3. **プロンプトの定義**:
   - 画像と、画像を処理して`plt`（matplotlib）を使って保存するPythonコードを生成するリクエストを含むメッセージを作成します。

4. **プロセッサの読み込み**:
   - `out_dir`ディレクトリで指定された事前学習済みモデルから`AutoProcessor`を読み込みます。このプロセッサはテキストと画像の入力を処理します。

5. **プロンプトの作成**:
   - `apply_chat_template`メソッドを使用して、メッセージをモデルに適したプロンプト形式に整形します。

6. **入力の処理**:
   - プロンプトと画像をモデルが理解できるテンソルに変換します。

7. **生成の引数を設定**:
   - 新しく生成されるトークンの最大数や、出力をサンプリングするかどうかなど、モデルの生成プロセスの引数を定義します。

8. **コードの生成**:
   - モデルが入力と生成引数に基づいてPythonコードを生成します。`TextStreamer`を使用して、プロンプトや特殊トークンをスキップしながら出力を処理します。

9. **出力**:
   - 生成されたコードが出力されます。このコードには、画像を処理してプロンプトで指定された通り保存するPythonコードが含まれます。

このデモでは、OpenVinoを使用して、事前学習済みモデルを活用し、ユーザーの入力と画像に基づいて動的にコードを生成する方法を示しています。

**免責事項**:  
この文書はAI翻訳サービス [Co-op Translator](https://github.com/Azure/co-op-translator) を使用して翻訳されています。正確性を追求していますが、自動翻訳には誤りや不正確な箇所が含まれる可能性があることをご承知ください。元の言語で記載された文書を正式な情報源として扱うべきです。重要な情報については、専門の人間による翻訳を推奨します。この翻訳の使用に起因する誤解や解釈の違いについて、当方は一切の責任を負いません。