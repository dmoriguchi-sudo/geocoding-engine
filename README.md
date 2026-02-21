# 🗺️ Geocoding Engine API

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python](https://img.shields.io/badge/python-3.11-blue.svg)](https://www.python.org/downloads/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.109-green.svg)](https://fastapi.tiangolo.com/)

住所リストを緯度・経度に変換する汎用APIサービスです。
Google Cloud Run上で動作し、GASや任意のHTTPクライアントから呼び出せます。

**🌐 本番URL**: https://geocoding-engine-959189601741.asia-northeast1.run.app
**📖 Swagger UI**: 上記URLにブラウザでアクセスするとインタラクティブな仕様書が表示されます。

---

## ✨ 特徴

- ✅ **一括変換**: 最大5,000件の住所を一度に変換
- ⚡ **並列処理**: 5スレッドで高速Geocoding
- 🔓 **汎用API**: GAS、Python、JavaScript等あらゆる環境から利用可能
- 📖 **自動ドキュメント**: Swagger UIで仕様を確認しながらテスト可能
- 🚀 **サーバーレス**: Google Cloud Runで自動スケール
- 🔑 **シンプルなI/O**: idと住所を投げるだけ、レスポンスはid付きJSON

---

## 🚀 クイックスタート

```bash
curl -X POST "https://geocoding-engine-959189601741.asia-northeast1.run.app/geocode" \
  -H "Content-Type: application/json" \
  -d '{
    "data": [
      {"id": "001", "address": "東京都千代田区丸の内1-1-1"},
      {"id": "002", "address": "東京都港区六本木6-1-5"}
    ]
  }'
```

---

## 📡 エンドポイント仕様

### `GET /`
Swagger UI（インタラクティブ仕様書）を表示

### `GET /health`
ヘルスチェック

### `POST /geocode`
住所 → 緯度経度変換

**リクエスト:**
```json
{
  "data": [
    { "id": "任意の識別子", "address": "住所文字列" }
  ]
}
```

**レスポンス:**
```json
{
  "results": [
    { "id": "001", "lat": 35.6812362, "lng": 139.7671248, "status": "SUCCESS" }
  ],
  "summary": {
    "total": 1, "success": 1, "not_found": 0,
    "no_address": 0, "error": 0, "elapsed_seconds": 1.23
  }
}
```

---

## 💡 使用例

### Google Apps Script (GAS)

```javascript
function geocodeSample() {
  const ENGINE_URL = 'https://geocoding-engine-959189601741.asia-northeast1.run.app/geocode';
  const payload = {
    data: [{ id: '001', address: '東京都千代田区丸の内1-1-1' }]
  };
  const response = UrlFetchApp.fetch(ENGINE_URL, {
    method: 'post',
    contentType: 'application/json',
    payload: JSON.stringify(payload)
  });
  const result = JSON.parse(response.getContentText());
  result.results.forEach(r => {
    if (r.status === 'SUCCESS') Logger.log(`${r.id}: ${r.lat}, ${r.lng}`);
  });
}
```

### Python

```python
import requests
url = "https://geocoding-engine-959189601741.asia-northeast1.run.app/geocode"
result = requests.post(url, json={"data": [{"id": "001", "address": "東京都千代田区丸の内1-1-1"}]}).json()
for r in result['results']:
    print(f"{r['id']}: {r['lat']}, {r['lng']} ({r['status']})")
```

---

## 💭 設計思想

- **GASの6分制限を回避**するためCloud Runで動作
- idを自由に設定できるため、呼び出し元が自由にマッピング可能
- フロントエンドなし — エンドポイントを叩くだけの汎用エンジン
- 複数用途（店舗スコアリング、コース分析、地図生成）で同一エンジンを使い回す

---

## ⚠️ 制限事項

| 項目 | 内容 |
|------|------|
| 推奨最大 | 5,000件 / リクエスト |
| タイムアウト | 3,600秒 |
| 並列スレッド数 | 5 |
| 処理速度目安 | 約1,000件/分（GAS比 約30倍） |

---

## 📊 ステータス一覧

| status | 意味 |
|--------|------|
| `SUCCESS` | 正常に座標取得 |
| `NOT_FOUND` | 住所が見つからない |
| `NO_ADDRESS` | 住所が空文字 |
| `ERROR: ...` | APIエラー |

---

## 📝 ライセンス

MIT License
