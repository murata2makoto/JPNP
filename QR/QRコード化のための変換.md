# PNPのQRコードによる表現

## * 目的

各受験者ごとにPNP設定ツールで作成したPNPデータを、PNPデータベースに登録して運用するのが、1EdTechで想定している運用形態である。しかし、PNPデータベースは個人情報、しかも障害についての個人情報の塊となってしまう。このようなデータベースを、個人情報保護をきちんと行って運用することは簡単ではない。広く社会の了承を得ること、PNPデータベースのセキュリティを保証すること、実際に運用することにはさまざまの困難が予想される。

こうした問題を回避するため、データベースを用いずに、使い捨てのQRコードでPNPを管理する方式を試みる。具体的には、各受験者に対してPNPをあらかじめ設定し、そこからQRコードを生成する。この時点でPNPデータは廃棄され、各受験者の手元にQRコードが残る。受検時には、このQRコードをコンピュータに付属するカメラで読み取らせてPNPデータを復元する。QRコードに個人名はいっさい含まれていないのでQRコードを紛失しても何の問題もない。

## * QRコードのどのバージョンが必要か

例題JSONファイルを三つのステップで圧縮する。

1. 引用符で囲まれていない空白と改行をすべて削除する
2. 頻出する文字列を制御文字に置き換える(stringReplacementDictionary.txtを参照）
3. Deflate圧縮

そのままのサイズ、改行と空白を取り除いた後のサイズ、そのままDeflate圧縮したときのサイズ、改行と空白を取り除いてからDeflate圧縮したときのサイズを次に示す。また、誤まり訂正Qのとき必要なQRコード（モデル2）のバージョンも示す。

| JSONファイル           | サイズ     | 必要なQRコードのバージョン |
| ---------------------- | ---------- | -------------------------- |
| そのまま               | 1004バイト |                            |
| 改行と空白の削除       | 620バイト  |                            |
| 頻出文字列の制御文字化 | 150バイト  | 18                         |
| Deflate圧縮            | 134バイト  | 17                         |

## 実装方式

以下はCoPilotが提案した実装方式。QRコードのバージョンや誤り訂正レベルをさらに考慮する必要がある。

Deflate圧縮を使って圧縮したデータからQRコードを生成することができます。以下にその一例を示します。まず、JavaScriptのpakoライブラリを利用してJSONデータを圧縮し、その圧縮データからQRコードを生成します。

### Deflate圧縮

javascript

```
function compressJSON(json) {
    const stringified = JSON.stringify(json);
    const compressed = pako.deflate(stringified, { to: 'string' });
    return compressed;
}
```

### QRコードの生成

次に、圧縮したデータからQRコードを生成するために、qrcode-generatorライブラリを使用します。

まず、ライブラリをインクルードするためにHTMLファイルに以下のスクリプトを追加します。

html

```
<script src="https://cdnjs.cloudflare.com/ajax/libs/pako/1.0.11/pako.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/qrcode-generator@1.4.4/lib/qrcode.js"></script>
```

次に、圧縮データのQRコードを生成する関数を実装します。

javascript

```
function createQR(data) {
    const qr = qrcode(0, 'L');
    qr.addData(data);  // 圧縮データをそのままQRコードに追加
    qr.make();

    document.getElementById('qrcode').innerHTML = qr.createImgTag();
}

const jsonData = { key: "value", array: [1, 2, 3], nested: { num: 4 } };
const compressedJSON = compressJSON(jsonData);

document.addEventListener("DOMContentLoaded", function() {
    createQR(compressedJSON);
});
```

### * HTMLコード

そして、HTMLファイルにQRコードを表示するための要素を追加します。

html

```
<!DOCTYPE html>

<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>QR Code Generator</title>
</head>
<body>
    <div id="qrcode"></div>
    <script src="path_to_your_js_file.js"></script>
</body>
</html>
```

これで、JSONデータをDeflate圧縮し、その圧縮データからQRコードを生成することができます。圧縮されたデータはQRコード内に含まれ、必要に応じて再度解凍して使用することができます。
