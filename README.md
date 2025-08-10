# HAGAKURE プログラミング塾 2025 — OpenAI API × Python 手書き実装

本リポジトリは、HAGAKURE プログラミング塾で使用した「OpenAI 公式 SDK（Responses API）を Python で手書き実装する」ための学習用スライド（`index.html`）と関連画像を含みます。スライド内には、最小実装から環境変数の扱い、エラーハンドリング、ストリーミング、簡易チャット実装、対話 CLI までのコード例が収録されています。

## できること（スライドに含まれるトピック）
- 公式 SDK（Responses API）での最小実装
- 環境変数（`.env`）での API キー管理
- 基本的なエラーハンドリング
- ストリーミング出力
- 会話履歴を貯める簡易チャット（クラス設計）
- 対話ループ（CLI 入力）

## スライドの見方
- `index.html` をブラウザで開くだけで閲覧できます
  - 直接ダブルクリックで開く、または簡易サーバで配信して閲覧

```bash
# 簡易HTTPサーバで配信（任意）
python -m http.server 8000
# ブラウザで http://localhost:8000/index.html を開く
```

## 事前準備（OpenAI Platform）
- OpenAI にサインアップ/ログイン（Billing の設定・支払い方法の登録を含む）
- API Keys ページで Secret Key を発行し、安全な場所に保存
- 本資料では `.env` に保存して使用する前提です

## Python SDK セットアップ（学習用）
スライドと同じコードをローカルで試す場合は、以下を参考にしてください。

```bash
pip install --upgrade openai python-dotenv
```

`.env` の例:

```bash
OPENAI_API_KEY=sk-xxxxx_your_key_here
OPENAI_MODEL=gpt-4o-mini
```

`config.py` の例:

```python
import os
from dotenv import load_dotenv

load_dotenv()
API_KEY = os.environ.get("OPENAI_API_KEY")
MODEL = os.environ.get("OPENAI_MODEL", "gpt-4o-mini")

if not API_KEY:
    raise RuntimeError("環境変数 OPENAI_API_KEY が設定されていません。")
```

最小実装（公式 SDK）:

```python
from openai import OpenAI

client = OpenAI(api_key=API_KEY)
response = client.responses.create(
    model=MODEL,
    input="日本語で、このSDKの最短使用例を1行で説明して。"
)
print(response.output_text)
```

ストリーミング（抜粋）:

```python
from openai import OpenAI

client = OpenAI(api_key=API_KEY)
with client.responses.stream(
    model=MODEL,
    input="200文字で自己紹介をストリーム出力してください。"
) as stream:
    for event in stream:
        if event.type == "response.output_text.delta":
            print(event.delta, end="", flush=True)
```

エラーハンドリング（抜粋）:

```python
from openai import AuthenticationError, RateLimitError, APIConnectionError, APIStatusError

try:
    ...
except AuthenticationError:
    print("認証エラー: APIキーを確認してください。")
except RateLimitError:
    print("レート制限: しばらく待って再試行してください。")
except APIConnectionError:
    print("接続エラー: ネットワークやDNSを確認してください。")
except APIStatusError as e:
    print(f"APIステータスエラー: {e.status_code} {e.response}")
```

チャットクラス（抜粋）:

```python
class ChatSession:
    def __init__(self, client: OpenAI, model: str):
        self.client = client
        self.model = model
        self.messages = []  # {role, content}

    def send(self, user_input: str) -> str:
        self.messages.append({"role": "user", "content": user_input})
        history_text = "\n".join(f"{m['role']}: {m['content']}" for m in self.messages)
        response = self.client.responses.create(model=self.model, input=history_text)
        assistant = response.output_text.strip()
        self.messages.append({"role": "assistant", "content": assistant})
        return assistant
```

## リポジトリ構成

```
hagakure_api_study2025/
├─ index.html        # 学習用スライド（本体）
└─ images/           # スライドで使用する画像
```

## Special Thanks
- 佐賀県産業スマート化センター様のコミュニティ支援により、ZOOM 有料プランをご提供いただきました。

## 注意事項
- API キーは公開しないでください。`.env` は Git にコミットしないでください。
- 本リポジトリの Python コードは学習用サンプルです。実運用では追加の安全対策・ロギング・監視等が必要です。

## ライセンス
このリポジトリには現時点で明示的なライセンスは含まれていません（必要に応じて追加してください）。

## 更新履歴
- 2025-08-10: 初版 README 追加