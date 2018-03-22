# atone payment

# 要件
https://redmine.savaway.co.jp/issues/31603

# atone 開発ドキュメント
| 項目 | 値 |
|:--|:--|
| URL | https://manual.atone.be/jp/ |
| ID | manual |
| PW | manualatone |

# 対応内容

## 開発画面
| システム | カテゴリ | 内容 | 備考 |
|:--|:--|:--|:--|
| マスタ管理 | 店舗詳細／決済設定 | ATONE枠の追加<br>店舗公開鍵、定期購入オプションの設定 |  |
| 店舗管理 | 決済方法設定一覧 | MigrationにてATONE用のpaymetsデータを追加 |  |
| 店舗管理 | 決済方法設定／注文金額制限 | ATONEの下限金額を1円に制御 |  |
| 店舗管理 | 決済方法設定／ATONE設定 | 画面追加<br>店舗秘密鍵の設定 |  |
| 店舗管理 | 未確認一覧 | ATONEの定期・頒布対応<br>オーソリ取得は非同期に処理依頼 |  |
| 店舗管理 | 売上計上待ち一覧 | ATONE用の画面追加<br>売上確定は非同期に処理依頼 |  |
| 店舗管理 | 受注CSVダウンロード | 決済方法コード、オンライン決済連携先コードにATONE決済の情報を出力 |  |
| フロント | 旧決済ステップ | ATONEの制限追加 |  |
| フロント | 新決済ステップ<br>（step2、step3） | 決済方法の出力にATONEを追加<br>定期・頒布は定期購入オプションによって制御 |  |
| フロント | 新決済ステップ<br>（step3） | atone指定のパラメータ出力、ボタン名の制御 |  |
| フロント | 新決済ステップ<br>（各モーダル編集時） | atone指定のパラメータを再出力、ボタン名の制御 | jsはria依頼 |
| フロント | 新決済ステップ<br>（認証後コールバックAjax） | 決済認証トークンを会員に紐付ける<br>済認証トークンを受注に紐付ける | jsはria依頼 |
| フロント | 新決済ステップ<br>（決済失敗コールバック） | 画面上部にエラーメッセージを出力してSTEP3を再表示 | jsはria依頼 |
| フロント | 新決済ステップ<br>（エラー発生コールバック） | 画面上部にエラーメッセージを出力してSTEP3を再表示 | jsはria依頼 |
| フロント | 新決済ステップ<br>（決済後コールバック） | ATONEの取引情報を取得。<br>受注情報の整合性チェック。<br>在庫引当して受注情報を生成<br>決済完了画面の表示<br>※受注不整合時はATONE受注をキャンセル | jsはria依頼 |

## 開発API（CARTSTAR側）
| システム | カテゴリ | 内容 | 備考 |
|:--|:--|:--|:--|
| API | 認証成功時Webhook | url : /v1/atone/member/[店舗ID]<br>決済認証トークンをCARTSTAR会員に紐付ける |  |
| API | 決済成功時Webhook | url : /v1/atone/order/[店舗ID]<br>CARTSTARとATONEの受注の整合性チェック、不整合時はATONE受注のキャンセル<br>Webhookが決済完了と同タイミングの場合は非同期依頼に変更する |  |

## 開発非同期処理
| システム | カテゴリ | 内容 | 備考 |
|:--|:--|:--|:--|
| 非同期 | 売上確定 | 売上確定処理、受注更新 |  |
| 非同期 | 定期購入 | オーソリ処理、次回受注生成 |  |

## ATONE連携
| システム | カテゴリ | 内容 | 備考 |
|:--|:--|:--|:--|
| 共通機能 | 取引参照API | ATONEの取引参照APIとの連携 | 注文完了後の受注生成時、決済成功時Webhookで利用 |
| 共通機能 | 売上確定API | ATONEの売上確定APIとの連携 | オーソリの売上確定時に利用 |
| 共通機能 | 課金払戻API | ATONEの課金払戻APIとの連携 | 受注キャンセル時、受注情報の不整合時に利用 |
| 共通機能 | 取引更新API | ATONEの取引更新APIとの連携 | 受注編集時に利用 |
| 共通機能 | 取引登録API | ATONEの取引登録APIとの連携 | 次回定期頒布のオーソリ取得時に利用 |

# DB関連
|論理テーブル名|物理テーブル名|用途|
|:--|:--|:--|
|外部アカウント連携|providers|ATONEとCARTSTARの会員情報の保持|
|ATONE受注情報|order_atones|受注に紐づくATONE受注情報の保持|

### membersテーブル　※カラム追加
#### カラム情報
|論理名|物理名|型|初期値|NULL許可|備考|
|:--|:--|:--|:--|:--|:--|
|UID|atone_uid|varchar(255)||○|ATONEの決済認証トークンを保存|

### order_atone_paymentsテーブル　※新規
#### カラム情報
|論理名|物理名|型|初期値|NULL許可|備考|
|:--|:--|:--|:--|:--|:--|
|ID|id|int(11)|||auto increment|
|受注ID|order_id|int(11)|||ordersテーブルのID|
|決済認証トークン|uid|varchar(255)|||ATONEの決済認証トークンを保存|
|取引オブジェクトID|object_id|varchar(30)|||ATONEの取引オブジェクトIDを保存|
|登録日時|created_at|datetime||||
|更新日時|updated_at|datetime||||

### Migration
paymentsテーブルにatone決済を1レコードINSERTする


# 設定関連

## config関連

* `config/params.php`

```php
  "import" => [
    "application.components.payments.stub.atone.*",
    // "application.components.payments.atone.*", // atone 本番用
  ],
  "payments" => [
    'atone' => [
      "name" => "atone",
      'atone_api' => 'https://ct-api.a-to-ne.jp/v1/', // テスト系
      // 'atone_api' => 'https://api.atone.be/v1/', // 本番系
      'atone_js' => 'https://ct-auth.a-to-ne.jp/v1/atone.js', // テスト系
      // 'atone_js' => 'https://auth.atone.be/v1/atone.js', // 本番系
      'webhook_api' => 'https://alpha-api.cartstar.net/v1', // テスト系
      // 'webhook_api' => 'https://api.corekago.jp/v1', // 本番系
    ],
  ],
```
