# Macなしで iPad テスト → App Store 販売まで進める手順

このフォルダは「Webアプリをネイティブの殻(Capacitor)に包み、ペン描画だけApple純正のPencilKitに差し替えた」App Store提出用プロジェクトです。ビルドはクラウドのMac(Codemagic)が行うので、**自分のMacは最後まで不要**です。

## 全体の流れ

iPadのブラウザだけで完結します。

1. **GitHubアカウント作成**(無料)→ このフォルダの中身をリポジトリとしてアップロード(Web画面の「Add file > Upload files」でOK)
2. **Apple Developer Program に登録**(年99ドル/ developer.apple.com)— TestFlightでの手元テストにも販売にも必須
3. **App Store Connect**(appstoreconnect.apple.com)で:
   - 「ユーザとアクセス > 統合 > App Store Connect API」でAPIキーを発行(Admin権限)。`.p8`ファイル・Key ID・Issuer IDを控える
   - 「マイApp > +」で新規App作成。Bundle IDは `capacitor.config.json` の `appId` と同じものを登録
4. **Codemagic**(codemagic.io)に無料登録 → GitHubリポジトリを接続:
   - Team settings > Integrations > App Store Connect に、手順3のAPIキーを **ASC_API_KEY** という名前で登録
   - `codemagic.yaml` と `capacitor.config.json` の `com.YOURNAME.pdfstudycards` を自分のBundle IDに書き換え
   - ビルド実行(署名証明書はCodemagicが自動生成・管理してくれます)
5. ビルド成功 → 自動でTestFlightにアップロード → **iPadのTestFlightアプリから自分のアプリをインストール**して試す
6. 満足したら App Store Connect で価格を設定し、スクリーンショット・説明文・**プライバシーポリシーURL**(必須)を用意して審査提出

## このプロジェクトの構成

```
www/                      Webアプリ本体(改善版ペン+PencilKit連携ブリッジ入り)
ios-extras/               PencilKitプラグイン(Swift + 登録用ObjC)
scripts/                  CIがSwiftファイルをXcodeプロジェクトへ登録するスクリプト
codemagic.yaml            クラウドビルド設定
capacitor.config.json     アプリID・名前
package.json              Capacitor依存
```

## アプリ内でのPencilKitの使い方

ネイティブアプリとして起動すると、描画ツールバーに「✍️ Pencil」ボタンが自動で現れます。タップすると現在のPDFページを背景に**Apple Notesと同じ描画キャンバス**が全画面で開き、純正ツールパレット(ペン・マーカー・消しゴム・定規・なげなわ)、筆圧・傾き、遅延ほぼゼロの予測描画、パームリジェクション、Pencilダブルタップでのツール切替がすべて使えます。「完了」で書き込みがページに合成されて戻ります。PWAやブラウザで開いた場合、このボタンは出ず従来のWebペンが使われます。

## 大事な注意

- **アプリ名について**: 「Anki」は既存の有名な学習アプリ(AnkiMobile)の名称で、この名前のまま販売すると商標クレームや審査リジェクトのリスクがあります。設定ではひとまず「PDF Study」にしてあります。販売前に必ず独自の名前に決めてください(`capacitor.config.json` の `appName`)。
- **審査(ガイドライン4.2)**: 「Webを包んだだけのアプリ」は弾かれることがありますが、本プロジェクトはPencilKitというネイティブ機能を核に据えているため主張材料があります。審査メモ欄に「Apple Pencil(PencilKit)によるPDF書き込みがコア機能」と書くのがおすすめです。
- **Swift部分は未ビルドのコード**です(私の環境ではiOSビルドができないため)。初回のクラウドビルドでエラーが出たら、ログをそのまま貼ってもらえれば修正版を出します。1〜2往復は見込んでください。
- **データはlocalStorage保存**です。ネイティブ化すると消えにくくなりますが、販売レベルでは将来的にファイルベース保存への移行を推奨します(これも対応可能です)。
- アプリアイコンは初回はCapacitorのデフォルトです。販売前に `@capacitor/assets` で差し替えます(やり方は聞いてください)。

## 費用まとめ

- GitHub / Codemagic: 無料枠でOK
- Apple Developer Program: 年99ドル(テストにも販売にも必須)
