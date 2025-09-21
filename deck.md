---
presentationID: 1WUVWmJb89XcPbkWwyXLABN8kYqWiB2lPxrl31xuTmY8
title: "OpenAI API × Python 手書き実装 - HAGAKUREプログラミング塾"
---
# OpenAI API × Python 手書き実装

HAGAKUREプログラミング塾　2025.08.17

![hero](images/hero-python.png)

---
## 講座の概要

- 公式SDK（Responses API）で最小実装
- 環境変数でAPIキー管理
- 基本のエラーハンドリング
- ストリーミング出力
- 会話を貯めるチャット（クラス）
- 対話ループ（input版）

---
## 環境確認

画面共有は見えていますか？

音声ははっきりと聞こえていますか？

チャットはSlackの方でお願いします。

---
## Special Thanks

佐賀県産業スマート化センターさまのコミュニティ支援にて、ZOOMの有料プランを提供いただき、利用させていただいております。

[https://www.saga-smart.jp/](https://www.saga-smart.jp/)

---
## 今日のゴール

APIキーを安全に管理し、最小→環境変数→エラーハンドリング→ストリーミングまで体験する

### ゴール
PythonでOpenAI APIの基礎を一気に！

---
## 事前準備：APIキーの取得

OpenAI Platform での手順

1. OpenAIにサインアップ/ログイン（[OpenAI Platform](https://platform.openai.com/)）
2. Billingの設定・支払い方法を登録（[Billing設定](https://platform.openai.com/settings/billing)）
3. API Keysページで「Create new secret key」（[API Keys](https://platform.openai.com/api-keys)）
4. 表示されたキーをコピーして安全な場所に保管
5. 本講座では `.env` に保存して使用

注意：APIキーは公開しない／ソース管理（Git等）にコミットしない

---
## セットアップ（pip）

```bash
pip install --upgrade openai python-dotenv
```

---
## SDK（環境変数なし）

まずは直書きで動かす

```python
from openai import OpenAI

client = OpenAI(api_key="sk-xxxxx_your_key_here")

response = client.responses.create(
    model="gpt-4o-mini",
    input="Write a short bedtime story about a unicorn."
)

print(response.output_text)
```

---
## APIキー管理：.env の例

```env
# .env
OPENAI_API_KEY=sk-xxxxx_your_key_here
OPENAI_MODEL=gpt-4o-mini
```

---
## APIキー管理：config.py

```python
import os
from dotenv import load_dotenv

load_dotenv()
API_KEY = os.environ.get("OPENAI_API_KEY")
MODEL = os.environ.get("OPENAI_MODEL", "gpt-4o-mini")

if not API_KEY:
    raise RuntimeError("環境変数 OPENAI_API_KEY が設定されていません。")
```

---
## SDK（環境変数あり）

.env + config.py を利用

```python
from openai import OpenAI
from config import API_KEY, MODEL

client = OpenAI(api_key=API_KEY)

response = client.responses.create(
    model=MODEL,
    input="日本語で、このSDKの最短使用例を1行で説明して。"
)

print(response.output_text)
```

---
## エラーハンドリング（公式SDK）

```python
from openai import OpenAI
from openai import AuthenticationError, RateLimitError, APIConnectionError, APIStatusError
from config import API_KEY, MODEL

client = OpenAI(api_key=API_KEY)
try:
    response = client.responses.create(
        model=MODEL,
        input="テスト"
    )
    print(response.output_text)
except AuthenticationError:
    print("認証エラー: APIキーを確認してください。")
except RateLimitError:
    print("レート制限: しばらく待って再試行してください。")
except APIConnectionError:
    print("接続エラー: ネットワークやDNSを確認してください。")
except APIStatusError as e:
    print(f"APIステータスエラー: {e.status_code} {e.response}")
except Exception as e:
    print(f"不明なエラー: {e}")
```

---
## ストリーミング（公式SDK）

```python
from openai import OpenAI
from config import API_KEY, MODEL

client = OpenAI(api_key=API_KEY)

with client.responses.stream(
    model=MODEL,
    input="200文字で自己紹介をストリーム出力してください。"
) as stream:
    for event in stream:
        if event.type == "response.output_text.delta":
            print(event.delta, end="", flush=True)
        elif event.type == "response.error":
            print("\nError:", event.error)
    final = stream.get_final_response()
    print("\n---\nFinal:", final.output_text)
```

---
## 会話を貯めるチャット（クラス）

履歴を文字列結合せず、roleごとのJSONメッセージ配列を渡す

```python
from openai import OpenAI
from typing import List, Dict

class ChatSession:
    def __init__(self, client: OpenAI, model: str):
        self.client = client
        self.model = model
        self.messages: List[Dict[str, str]] = []  # {role, content}

    def send(self, user_input: str) -> str:
        self.messages.append({"role": "user", "content": user_input})
        response = self.client.responses.create(model=self.model, input=self.messages)
        assistant = response.output_text.strip()
        self.messages.append({"role": "assistant", "content": assistant})
        return assistant

# 使い方
from config import API_KEY, MODEL
client = OpenAI(api_key=API_KEY)
chat = ChatSession(client, MODEL)
print(chat.send("自己紹介して"))
print(chat.send("今の回答を短く3箇条で"))
print(chat.send("最後に一言メッセージを"))
```

---
## 対話ループ（input版）

```python
# 上の ChatSession を使って、対話CLIを実装
from openai import OpenAI
from config import API_KEY, MODEL

client = OpenAI(api_key=API_KEY)
chat = ChatSession(client, MODEL)

print("対話を開始します。終了は 'exit' または 'quit'。")
while True:
    try:
        user_input = input("You> ").strip()
    except (EOFError, KeyboardInterrupt):
        print("\n終了します。")
        break
    if user_input.lower() in ("exit", "quit"):
        print("終了します。")
        break
    reply = chat.send(user_input)
    print("Assistant>", reply)
```


