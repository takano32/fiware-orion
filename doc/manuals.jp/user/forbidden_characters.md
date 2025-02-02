# 禁止されている文字 (Forbidden characters)

## 禁止されている文字

いくつかの状況でスクリプト・インジェクション攻撃を回避するために、次の文字はすべてのリクエストで禁止されています (例えば Context Broker と同じホスト内の同じ場所にある Web サーバとのクロス・ドメイン) :

-   &lt;
-   &gt;
-   "
-   '
-   =
-   ;
-   (
-   )

これらを使用しようとすると、次のような 400 Bad Request レスポンスが発生します :

    {
        "error": "BadRequest",
        "description": "Invalid characters in attribute type"
    }

アプリケーションがこれらの文字を使用する必要がある場合は、Orion にリクエストを送信する前に、禁止文字を含まないスキームを使用してエンコードする必要があります。

[URL エンコーディング](http://www.degraeve.com/reference/urlencoding.php)は、有効なエンコーディング方法です。 ただし、"%" キャラクタ自体をエンコードする必要があるため、エンティティ ID や属性名などの API URL に表示される可能性のあるフィールドの使用はお勧めしません。 たとえば、エンティティ ID として "E<01>" を使用する場合、その URL エンコードは、"E%3C01%3E" となります。

エンティティの情報取得オペレーションなどで、このエンティティ ID を URL 中で使用するために、以下が使用されます。"%25" は "%" のエンコーディングであることに注意してください。

```
GET /v2/entities/E%253C01%253E
```

### 例外

上記の制限が適用されない例外がいくつかあります。特に、次のフィールドでは :

* URL パラメータ `q` は、シンプル・クエリ言語が必要とする特殊文字を許可します
* URL パラメータ `mq` は、シンプル・クエリ言語が必要とする特殊文字を許可します
* URL パラメータ `georel` と `coords` は `;` を許可します
* 属性タイプとして "TextUnrestricted" を使用する属性値。セキュリティに影響する
  可能性があるため、自己責任で使用してください!

## ID フィールドの特定の制限事項

NGSIv2 では、ID フィールド (エンティティ ID/型, 属性名/型, メタデータ名/型など) の構文制限が導入されています。これについては、[NGSIv2 仕様](http://telefonicaid.github.io/fiware-orion/api/v2/stable/)の "Field syntax restrictions" のセクションで説明しています。

## カスタム・ペイロードとヘッダの特別扱い

NGSIv2 はカスタム通知を生成するサブスクリプションのためのテンプレート・メカニズムを
提供します ([NGSIv2 仕様](http://telefonicaid.github.io/fiware-orion/api/v2/stable/)の
"Custom notifications" を参照)。禁止された文字の制限は、POST/v2/subscription または
GET/v2/subscriptions のような、NGSIv2 API オペレーションの `httpCustom.payload`
フィールドに適用されます。`httpCustom.headers` で同じ制限がヘッダ値に適用されます。

ただし、通知時に、`httpCustom.payload` または `httpCustom.headers` の値で URL
エンコードされた文字はすべてデコードされます。

例 :

特定のサブスクリプション中の次のような `notification.httpCustom` オブジェクトを考えてみましょう。

```
"httpCustom": {
  "url": "http://foo.com/entity/${id}",
  "headers": {
    "Content-Type": "text/plain",
    "Authorization": "Basic ABC...ABC%3D%3D"
  },
  "method": "PUT",
  "qs": {
    "type": "${type}"
  },
  "payload": "the value of the %22temperature%22 attribute %28of type Number%29 is ${temperature}"
}
```

上記ペイロード値は、この文字列の URL エンコードされたバージョンであることに注意してください :
`(type Number の) "temperature" 属性の値は ${temperature}` です。`"Basic ABC...ABC%3D%3D"`
は、この文字列の URL エンコードされたバージョンであることに注意してください: `"Basic ABC...ABC=="`

さて、NGSIv2 実装がこのサブスクリプションに関連する通知をトリガすることを考えてみましょう。通知データは、id が `DC_S1-D41`、型が `Room` で、値が 23.4 の temperature という名前の属性を含むエンティティ用です。テンプレートを適用した結果の通知は次のようになります :

```
PUT http://foo.com/entity/DC_S1-D41?type=Room
Authorization: "Basic ABC...ABC=="
Content-Type: application/json 
Content-Length: 65

the value of the "temperature" attribute (of type Number) is 23.4
```
