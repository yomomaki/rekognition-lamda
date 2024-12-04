# rekognition-lamda
 
### こちらの動画で扱ったソースになります
[動画リンク](https://youtu.be/vAD4Zqx0IeE)

## バージョン
主なもの
- Lambda ランタイムPython 3.13
 
### Lambdaコード
```
import json
import boto3
import os

# SDK
rekognitionClient = boto3.client('rekognition')
s3 = boto3.resource('s3')

# Rekognition お試しLambda
def lambda_handler(event, context):
         
    image = None
    # バケットは環境変数から
    bucket = os.environ['buketName']
    
    # 取得するオブジェクト名は引数から取るようにする
    key = event['object'] #'test3.png'

    try:
        # S3から画像取得
        s3_object = s3.Object(bucket, key)
        image = s3_object.get()['Body'].read()
        
        # Rekognitionで画像判定
        res = rekognitionClient.detect_moderation_labels(Image={'Bytes': image})
        # print('-----resall:', res)
        print('-----resLabelName:', [label['Name'] for label in res['ModerationLabels']])
        return {
           "body": res
        }

    except Exception as e:
        print('------error: ',e)
        return 400
```

### S3バケット
あらかじめS3バケットを作成しておき、Lambdaの設定タブに「環境変数」から設定しておくこと
```
key： buketName
value: {S3バケット名}
```

## Lambdaに設定するIAMロール
以下のIAMポリシーを設定したロールを作成してLambdaの実行ロールに設定します。  

AWS 管理(マネージドのポリシー↓2つと)  
・ AmazonS3ReadOnlyAccess (S3から画像を取得する権限)  
・ AWSLambdaBasicExecutionRole(CloudWatchにログ出力させるロール)

LambdaからRekognitionを使う権限のIAMロールは以下のIAMポリシーを設定したロールとして作成します。
```
{
  "Version": "2012-10-17",
  "Statement": [
      {
          "Effect": "Allow",
          "Action": [
              "rekognition:DetectModerationLabels"
          ],
          "Resource": "*"
      }
  ]
}
```
LambdaからRekognitionのDetectModerationLabels（適切なコンテンツの検出）を行う権限  
Labelの検出をする場合は```rekognition:DetectLabels``` を追加する


