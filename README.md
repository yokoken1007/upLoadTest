import json
import requests
import re
import boto3

def lambda_handler(event, context):
    
    # BitriseのAPIトークンとApp Slugを設定
    bitrise_token = 'IMTCRipnRVnFzJ1sSUyymAtEU0XuJYGw-fl9E95pBS_3OYIutbBjJm8k2zf1H9qCcxPFNYR3sLzNJ-jXSoJtgg'
    app_slug = '323cf1e3-f93c-49b2-9347-470950f0e4dc'

    # BitriseのAPIエンドポイントを構築
    bitrise_api_url = f'https://api.bitrise.io/v0.1/apps/{app_slug}/builds'
    
    if 'Records' in event and len(event['Records']) > 0:
        if event['Records'][0].get('eventSource') == 'aws:codecommit':
            # CodeCommitのトリガーで呼び出された場合の処理
            print("CodeCommitのトリガーで呼び出されました")
            
            # イベント情報からブランチ名を取得
            branch_name = event['Records'][0]['codecommit']['references'][0]['ref']
            print(f"ブランチ名は:{branch_name}")
            
            branch_name_simple = branch_name.split("refs/heads/")[1]
            print(f"変更ブランチ名は {branch_name_simple} です")
            
            # ビルドトリガーペイロードを作成
            payload = {
                'hook_info': {
                    'type': 'bitrise'
                },
                'build_params': {
                    'branch': f'{branch_name_simple}'
                }
            }
            
            
        elif event['Records'][0].get('EventSource') == 'aws:sns':
            # AWS SNSの通知で呼び出された場合の処理
            print("AWS SNSの通知で呼び出されました")
            
            codecommit_client = boto3.client('codecommit')
    
            if 'Records' in event and len(event['Records']) > 0:
                
                record = event['Records'][0]

                # レコードからメッセージ部分を取得
                message = record['Sns']['Message']
                
                # メッセージを辞書オブジェクトに変換
                message_dict = json.loads(message)
                
                # リポジトリ名を取得
                repository_name = message_dict['detail']['repositoryNames'][0]
                
                # ブランチ名を取得
                branch_name = message_dict['detail']['sourceReference']
                destination_branch_name = message_dict['detail']['destinationReference']
                # ブランチ名の「refs/heads/」を削除
                branch_name_simple = branch_name.split("refs/heads/")[1] 
                destination_branch_name_simple = destination_branch_name.split("refs/heads/")[1] 
                
                # sourceCommitを取得
                source_commit = message_dict['detail']['sourceCommit']
                
                # プルリクエストIDを取得
                pull_request_id = message_dict['detail']['pullRequestId']
                
                # print("Repository Name:", repository_name)
                # print("branch_name:", branch_name)
                # print("destination_branch_name:", destination_branch_name)
                # print("Source Commit:", source_commit)
                # print("Pull Request ID:", pull_request_id)
                
                # ビルドトリガーペイロードを作成
                payload = {
                    "hook_info":{
                        "type":"bitrise"
                    },
                    "build_params": {
                        "branch": f"{branch_name_simple}",
                        "branch_dest": f"{destination_branch_name_simple}",
                        "commit_hash": f"{source_commit}",
                        "pull_request_id": f"{pull_request_id}"
                    }
                }

            else:
                print("イベント情報が不正です") 
            
        else:
            # その他の場合の処理
            print("その他のトリガーで呼び出されました")
    else:
        # イベントが存在しない場合の処理
        print("イベントが存在しません")
        
    
    print("payloadは:", payload)

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
