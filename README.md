import json
import requests

def lambda_handler(event, context):

    if 'Records' in event and len(event['Records']) > 0:
        if event['Records'][0].get('eventSource') == 'aws:codecommit':
            # CodeCommitのトリガーで呼び出された場合の処理
            print("CodeCommitのトリガーで呼び出されました")
        elif event['Records'][0].get('EventSource') == 'aws:sns':
            # AWS SNSの通知で呼び出された場合の処理
            print("AWS SNSの通知で呼び出されました")
        else:
            # その他の場合の処理
            print("その他のトリガーで呼び出されました")
    else:
        # イベントが存在しない場合の処理
        print("イベントが存在しません")


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
       
