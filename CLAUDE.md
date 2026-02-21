# CLAUDE.md - geocoding-engine

## プロジェクト概要
住所リストを緯度・経度に変換する汎用APIサービス。
Google Cloud Run上で動作し、GASや任意のHTTPクライアントから呼び出せる。

## 本番URL
https://geocoding-engine-959189601741.asia-northeast1.run.app/docs

## 技術スタック
- Python 3.11 / FastAPI
- Google Cloud Run（サーバーレス）
- Google Maps Geocoding API
- 並列処理: 5スレッド

## 主な仕様
- 最大5,000件/リクエスト（ソフトリミット）
- タイムアウト: 3,600秒（1時間）
- 処理速度: 約1,000〜1,400件/分（GAS比約30倍）

## エンドポイント
- `GET /` → Swagger UI
- `GET /health` → ヘルスチェック
- `POST /geocode` → 住所→緯度経度変換

## リクエスト/レスポンス形式
```json
// リクエスト
{ "data": [{ "id": "001", "address": "東京都千代田区丸の内1-1-1" }] }

// レスポンス
{ "results": [{ "id": "001", "lat": 35.68, "lng": 139.76, "status": "SUCCESS" }] }
```

## ステータス値
| status | 意味 |
|--------|------|
| SUCCESS | 正常取得 |
| NOT_FOUND | 住所が見つからない |
| NO_ADDRESS | 住所が空 |
| ERROR | APIエラー |

## デプロイ方法
```bash
gcloud run deploy geocoding-engine \
  --source . \
  --region asia-northeast1 \
  --allow-unauthenticated \
  --memory 512Mi \
  --timeout 3600 \
  --set-env-vars GOOGLE_MAPS_API_KEY=YOUR_API_KEY
```

## 環境変数
- `GOOGLE_MAPS_API_KEY` : Google Maps Geocoding APIキー

## 注意事項
- Google Maps API の料金が発生する
- 大量データは分割して投げる方が推奨
