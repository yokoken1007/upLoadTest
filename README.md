import json
import requests

def lambda_handler(event, context):
    # BitriseのAPIトークンとApp Slugを設定
    bitrise_token = 'IMTCRipnRVnFzJ1sSUyymAtEU0XuJYGw-fl9E95pBS_3OYIutbBjJm8k2zf1H9qCcxPFNYR3sLzNJ-jXSoJtgg'
    app_slug = '323cf1e3-f93c-49b2-9347-470950f0e4dc'

    # BitriseのAPIエンドポイントを構築
    bitrise_api_url = f'https://api.bitrise.io/v0.1/apps/{app_slug}/builds'

    # ビルドトリガーペイロードを作成
    payload = {
        'hook_info': {
            'type': 'bitrise'
        },
        'build_params': {
            'branch': 'main'
        }
    }

    # リクエストヘッダーを設定
    headers = {
        'Content-Type': 'application/json',
        'Authorization': f'token {bitrise_token}'
    }

    # POSTリクエストを送信
    response = requests.post(bitrise_api_url, json=payload, headers=headers)

    # レスポンスを処理
    if response.status_code == 201:
        return 'Build triggered successfully'
    else:
        return f'Failed to trigger build. Status code: {response.status_code}, Error: {response.text}'
