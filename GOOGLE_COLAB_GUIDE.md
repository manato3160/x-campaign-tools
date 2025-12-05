# Google Colab連携ガイド

このガイドでは、Xキャンペーン自動抽選ツールをGoogle Colabで実行する方法を説明します。

## 概要

Google Colabを使用することで、以下のメリットがあります：

- **無料のクラウド環境**: ローカル環境のセットアップが不要
- **高性能リソース**: Colabの無料GPU/TPUリソースを活用可能
- **Google Drive連携**: CSVファイルの保存・共有が簡単
- **ブラウザから実行**: どこからでもアクセス可能

## セットアップ手順

### 1. Google Colabでノートブックを開く

1. [Google Colab](https://colab.research.google.com/)にアクセス
2. 「ファイル」→「ノートブックをアップロード」を選択
3. `Xキャンペーン自動抽選_GoogleColab.ipynb`をアップロード

または、Google Driveにノートブックを保存して、右クリック→「アプリで開く」→「Colaboratory」を選択

### 2. 依存パッケージのインストール

最初のセルを実行して、必要なパッケージをインストールします：

```python
!pip install -q xai-sdk>=1.3.1 python-dotenv>=1.0.0 pydantic>=2.0.0 pandas>=2.0.0 openpyxl>=3.1.0
```

### 3. XAI_API_KEYの設定

**重要**: xAI APIキーを取得して設定する必要があります。

1. [xAI Console](https://console.x.ai/)にアクセス
2. APIキーを取得
3. ノートブックの「環境変数の設定」セルで、APIキーを入力

```python
import os
from getpass import getpass

api_key = getpass('XAI_API_KEYを入力してください: ')
os.environ['XAI_API_KEY'] = api_key
```

**セキュリティ注意事項**:
- APIキーは`getpass`を使用して入力するため、画面に表示されません
- ノートブックを共有する場合は、APIキーが含まれていないことを確認してください

### 4. Google Driveのマウント（オプション）

Google DriveにCSVファイルを保存している場合、Driveをマウントします：

```python
from google.colab import drive
drive.mount('/content/drive')
```

マウント後、Drive内のファイルは`/content/drive/MyDrive/`からアクセスできます。

### 5. CSVファイルのアップロード

ノートブックの「CSVファイルのアップロード」セルを実行して、CSVファイルをアップロードします。

**対応形式**:
- **旧形式**: `handle`列を含むCSV
- **新形式**: A列（ユーザーID）とB列（URL）が4行目以降から始まるCSV

### 6. 抽選処理の実行

「抽選処理の実行」セルで、以下のパラメータを設定できます：

```python
LIMIT = None  # テスト用: 10件のみ処理する場合は 10 を設定
WORKERS = 50  # 並列処理のワーカー数
CHUNK_SIZE = 200  # 1チャンクあたりの処理件数
CAMPAIGN_KEYWORD = ""  # キャンペーンキーワード（オプション）
```

**推奨設定**:
- **テスト実行**: `LIMIT = 10`, `WORKERS = 20`
- **本番実行**: `LIMIT = None`, `WORKERS = 50`, `CHUNK_SIZE = 200`

### 7. 結果のダウンロード

処理完了後、「結果ファイルのダウンロード」セルを実行して、CSVファイルをダウンロードします。

## よくある質問（FAQ）

### Q: APIキーはどこで取得できますか？

A: [xAI Console](https://console.x.ai/)で取得できます。アカウント登録が必要です。

### Q: 処理速度を上げるにはどうすればいいですか？

A: `WORKERS`の値を増やしてください（例: 50 → 100）。ただし、APIレート制限に達する可能性があります。

### Q: メモリ不足エラーが発生しました

A: `CHUNK_SIZE`を小さくしてください（例: 200 → 100）。または、`LIMIT`を設定して処理件数を制限してください。

### Q: CSVファイルが読み込めません

A: 以下の点を確認してください：
- CSVファイルがUTF-8エンコードであること
- ファイル形式が正しいこと（旧形式または新形式）
- ファイルが正しくアップロードされていること

### Q: Google Driveのファイルを使用したい

A: Driveをマウント後、以下のようにファイルパスを指定できます：

```python
csv_filename = "/content/drive/MyDrive/path/to/your/file.csv"
handles = load_handles_from_csv(csv_filename)
```

### Q: 処理を中断したい

A: Colabの「ランタイム」→「実行の中断」を選択してください。

### Q: セッションがタイムアウトしました

A: Colabの無料版では、90分間非アクティブでセッションがタイムアウトします。処理中は定期的にセルを実行するか、Colab Proを検討してください。

## トラブルシューティング

### APIキーエラー

```
エラー: XAI_API_KEYが環境変数に設定されていません。
```

**解決方法**:
1. 「環境変数の設定」セルを再実行
2. APIキーが正しく入力されているか確認
3. [xAI Console](https://console.x.ai/)でAPIキーが有効か確認

### CSV読み込みエラー

```
エラー: CSVファイルのエンコーディングを検出できませんでした
```

**解決方法**:
1. CSVファイルをUTF-8エンコードで保存
2. Excelで開いて「名前を付けて保存」→「CSV UTF-8（コンマ区切り）」を選択

### APIレート制限エラー

```
エラー: APIレート制限に達しました
```

**解決方法**:
1. `WORKERS`の値を減らす（例: 50 → 20）
2. しばらく待ってから再実行
3. xAI APIの利用制限を確認

### メモリ不足エラー

```
エラー: メモリ不足
```

**解決方法**:
1. `CHUNK_SIZE`を小さくする（例: 200 → 100）
2. `LIMIT`を設定して処理件数を制限
3. Colabのランタイムタイプを「高RAM」に変更（Colab Pro）

## パフォーマンス最適化

### 処理速度の目安

- **デフォルト設定**（50ワーカー、200件チャンク）: 約1時間で10,000件処理
- **高速設定**（100ワーカー、500件チャンク）: 約30-45分で10,000件処理
- **保守的設定**（20ワーカー、100件チャンク）: 約2-3時間で10,000件処理

### 推奨設定

**大量データ処理（10,000件以上）**:
```python
WORKERS = 50
CHUNK_SIZE = 200
```

**中規模データ処理（1,000-10,000件）**:
```python
WORKERS = 30
CHUNK_SIZE = 100
```

**小規模データ処理（100-1,000件）**:
```python
WORKERS = 20
CHUNK_SIZE = 50
```

**テスト実行（10-100件）**:
```python
LIMIT = 10
WORKERS = 10
CHUNK_SIZE = 10
```

## セキュリティに関する注意事項

1. **APIキーの管理**:
   - APIキーは`getpass`を使用して入力してください
   - ノートブックを共有する前に、APIキーが含まれていないことを確認してください
   - 不要になったAPIキーは無効化してください

2. **データの取り扱い**:
   - 個人情報を含むCSVファイルの取り扱いに注意してください
   - Google Driveに保存する場合は、適切な共有設定を行ってください

3. **セッション管理**:
   - 処理完了後は、セッションを終了してください
   - ノートブックを共有する場合は、機密情報が含まれていないことを確認してください

## 参考リンク

- [xAI Console](https://console.x.ai/)
- [xAI Python SDK GitHub](https://github.com/xai-org/xai-sdk-python)
- [xAI Documentation](https://docs.x.ai)
- [Google Colab](https://colab.research.google.com/)

## サポート

問題が発生した場合、以下の情報を含めてお問い合わせください：

1. エラーメッセージの全文
2. 実行したセルの内容
3. CSVファイルの形式（旧形式/新形式）
4. 処理件数と設定パラメータ
