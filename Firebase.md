# Firebase

## Firestore Database

### Firestore Database Error

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
