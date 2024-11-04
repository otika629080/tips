# Firebase

## Firestore Database

### Firestore Database Error

以下のようなコードをクライアントサイドに書いてエラーの種類を把握すると良い。
```ts
const handleGoogleLogin = async () => {
	try {
    		// Firestoreへの書き込み処理など
	} catch (error) {
    		alert(error)
	}
```

#### FirebaseError: [code=permission-denied]: Missing or insufficient permissions. (書き込みができない場合)
管理コンソール > Firestore Database > ルールの順に開き、以下を確認する。

デフォルトのテスト環境を構築すると読み書きの設定がfalseになっている可能性があるので、それを修正する。

```js
rules_version = '2';

service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if request.time < timestamp.date(xxxx, xx, x); // 時刻のパラメータを確認する
    }
  }
}
```

#### 'The default Firebase app does not exist. Make sure you call initializeApp() before using any of the Firebase services.'
エラーの原因は、おそらく`FB_PRIVATE_KEY`が未設定であるためです。Firebase Admin SDKを初期化するには、サービスアカウントの認証情報が必要で、これらのキーが欠けていると、Firebaseに接続できません。

###### 1. `FB_CLIENT_EMAIL`と`FB_PRIVATE_KEY`の取得方法
これらの情報は、Firebaseプロジェクトの**サービスアカウント**から取得できます。

###### サービスアカウントJSONファイルの取得手順：
1. **Firebaseコンソール**にアクセスし、プロジェクトを選択します。
2. 左側のメニューから **[プロジェクトの設定]** を開きます。
3. **[サービスアカウント]** タブに移動します。
4. **[新しい秘密鍵を生成]** ボタンをクリックして、JSONファイルをダウンロードします。

このJSONファイルの中に、以下のような形式で認証情報が含まれています：

```json
{
  "type": "service_account",
  "project_id": "your-project-id",
  "private_key_id": "some-key-id",
  "private_key": "-----BEGIN PRIVATE KEY-----\nYOUR_PRIVATE_KEY\n-----END PRIVATE KEY-----\n",
  "client_email": "your-service-account@your-project-id.iam.gserviceaccount.com",
  "client_id": "your-client-id",
  ...
}
```
- **`private_key`** の値を `.env` の `FB_PRIVATE_KEY` に設定します。  
  この値には改行が含まれているため、改行は `\n` に置き換えます（例：`-----BEGIN PRIVATE KEY-----\nYOUR_PRIVATE_KEY\n-----END PRIVATE KEY-----`）。

###### `.env`ファイルに設定例
```plaintext
FB_PROJECT_ID=your-project-id
FB_CLIENT_EMAIL=your-service-account@your-project-id.iam.gserviceaccount.com
FB_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\nYOUR_PRIVATE_KEY\n-----END PRIVATE KEY-----\n"
```

###### 注意点
- `FB_PRIVATE_KEY`を`.env`ファイルに設定する際、`" "`（ダブルクォーテーション）で囲むと、改行文字が正しく認識されることがあります。
- 認証情報を安全に保管し、アクセス権を適切に設定してください。
