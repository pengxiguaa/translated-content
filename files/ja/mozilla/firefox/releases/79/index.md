---
title: Firefox 79 for developers
slug: Mozilla/Firefox/Releases/79
tags:
  - '79'
  - Firefox
  - Mozilla
  - Release
translation_of: Mozilla/Firefox/Releases/79
---
{{FirefoxSidebar}}

このページでは、開発者に影響する Firefox 79 の変更点をまとめています。Firefox 79 は、2020 年 7 月 28 日にリリースされました。

Mozilla hacks の記事「[Firefox 79: The safe return of shared memory, new tooling, and platform updates](https://hacks.mozilla.org/2020/07/firefox-79/)」もご覧ください。

## ウェブ開発者向けの変更点一覧

### 開発者ツール

#### コンソール

- レスポンスコードが 400-499 および 500-599 の範囲にあるネットワークメッセージを、エラーとみなすようになりました。また、[要求や XHR のフィルターが無効であっても](/ja/docs/Tools/Web_Console/Console_messages#Filtering_by_category) 表示するようになりました ({{bug(1635460)}})。
- (ブラウザーや拡張機能によって) ブロックされた要求のネットワークメッセージが、[コンソール](/ja/docs/Tools/Web_Console/Console_messages) で "禁止" アイコンがつくようになりました ({{bug(1629875)}})。

#### デバッガー

- [ソースファイルを "ブラックボックス化"](/ja/docs/Tools/Debugger/How_to/Black_box_a_source) を、ソースファイルを "無視" と呼ぶようになりました ({{bug(1642811)}})。
- [例外](/ja/docs/Tools/Debugger/How_to/Breaking_on_exceptions) でインラインプレビューが可能になりました ({{bug(1581708)}})。
- 監視式やスコープのセクションにある項目にマウスポインターを載せると、それらの値を表示するツールチップが現れるようになりました ({{bug(1631545)}})。
- [コールスタックセクション](/ja/docs/Tools/Debugger/UI_Tour#Call_stack) のコンテキストメニューの項目に、現在のスタックフレームをはじめから実行する **フレームを再実行** を追加しました ({{bug(1594467)}})。

#### その他のツール

- 新たに [アプリケーションパネル](/ja/docs/Tools/Application) が使用可能になりました。まずは [service worker](/ja/docs/Web/API/Service_Worker_API) および [ウェブアプリマニフェスト](/ja/docs/Web/Manifest) の調査やデバッグの機能を提供します。
- ネットワークモニターのメッセージタブを、[応答タブ](/ja/docs/Tools/Network_Monitor/request_details#Response_tab) に統合しました ({{bug(1636421)}})。
- アクセシビリティインスペクターが、タブにアクセスすると自動的に有効化します。明示的に有効化することが不要になりました ({{bug(1602075)}})。
- [レスポンシブデザインモード](/ja/docs/Tools/Responsive_Design_Mode#Controlling_Responsive_Design_Mode) でタッチシミュレーションを有効にしたとき、マウスドラッグのイベントをタッチ & ドラッグまたはスワイプのイベントとして解釈するようになりました ({{bug(1621781)}})。
- [リモートデバッグ](/ja/docs/Tools/about:debugging#Connecting_to_a_remote_device) で、リモートブラウザーのナビゲーションを支援するための **戻る** および **進む** ボタンを URL バーに追加しました ({{bug(1639425)}})。

### HTML

- [`<iframe>`](/ja/docs/Web/HTML/Element/iframe) 要素の `sandbox` 属性で `allow-top-navigation-by-user-activation` トークンをサポートしました ({{bug(1359867)}})。
- [`<a>`](/ja/docs/Web/HTML/Element/a) および [`<area>`](/ja/docs/Web/HTML/Element/area) 要素で `target="_blank"` を設定すると、`rel="noopener"` も指定したときと同じ動作を暗黙的に提供するようになりました ({{bug(1522083)}})。

### CSS

- 外部スタイルシートが、ドキュメントグループごとにキャッシュされるようになりました ({{bug(1599160)}})。同じオリジンのページへ移動するとき、Firefox はキャッシュされたスタイルシートの検索や再検証を最小限にします。単純な再読み込み (例えば&#x20;

  <kbd>F5</kbd>

  ) では、キャッシュされた CSS ファイルを再検証しません。現在のバージョンのスタイルシートを読み込むには、キャッシュをバイパスしてページを再読み込みします (

  <kbd>Cmd</kbd>

  /

  <kbd>Ctrl</kbd>

  &#x20;\+&#x20;

  <kbd>F5</kbd>

  )。

#### 廃止

- メディア特性 [`prefers-color-scheme`](/ja/docs/Web/CSS/@media/prefers-color-scheme) の値 `no-preference` が、[media queries 仕様書](https://drafts.csswg.org/mediaqueries-5/#descdef-media-prefers-color-scheme) および Firefox から削除されました ({{bug(1643656)}})。

### JavaScript

- {{jsxref("SharedArrayBuffer")}} を、post-Spectre-safe な方法で再び有効化しました。クロスオリジン分離のサイトで使用できます ({{bug(1619649)}})。

  - サイトをクロスオリジン分離にするには、新たに {{HTTPHeader("Cross-Origin-Embedder-Policy")}} (COEP) および {{HTTPHeader("Cross-Origin-Opener-Policy")}} (COOP) ヘッダーを設定することが必要です。

- {{jsxref("Promise.any()")}} が使用可能になりました ({{bug(1599769)}})。
- {{jsxref("WeakRef")}} オブジェクトを実装しました ({{bug(1639246)}})。
- [Logical assignment operators](https://github.com/tc39/proposal-logical-assignment) をサポートしました ({{bug(1639591)}})。

  - [Logical nullish assignment (`??=`)](/ja/docs/Web/JavaScript/Reference/Operators/Logical_nullish_assignment)
  - [Logical AND assignment (`&&=`)](/ja/docs/Web/JavaScript/Reference/Operators/Logical_AND_assignment)
  - [Logical OR assignment (`||=`)](/ja/docs/Web/JavaScript/Reference/Operators/Logical_OR_assignment)

- {{jsxref("Atomics")}} オブジェクトが、共有されていないメモリーでも動作するようになりました ({{bug(1630706)}})。
- [`Intl.DateTimeFormat()` コンストラクター](/ja/docs/Web/JavaScript/Reference/Global_Objects/Intl/DateTimeFormat/DateTimeFormat) で、`dateStyle` および `timeStyle` オプションをサポートしました ({{bug(1557718)}})。
- [`Intl.NumberFormat()` コンストラクター](/ja/docs/Web/JavaScript/Reference/Global_Objects/Intl/NumberFormat/NumberFormat) で、さらに多くの表記法をサポートしました ({{bug(1413504)}})。

### HTTP

- 新たな {{HTTPHeader("Cross-Origin-Embedder-Policy")}} (COEP) および {{HTTPHeader("Cross-Origin-Opener-Policy")}} (COOP) ヘッダー使用する、クロスオリジン分離を実装しました。これは {{jsxref("SharedArrayBuffer")}} オブジェクトや {{domxref("Performance.now()")}} の制限されていないタイマーといった、特定の機能へのアクセスを可能にします。

### API

#### DOM

- [`FileReader`](/ja/docs/Web/API/FileReader) インターフェイスの [`loadstart` イベント](/ja/docs/Web/API/FileReader/loadstart_event) が、仕様書に従って非同期に発生するようになりました ({{bug(1502403)}})。
- {{domxref("CanvasPattern.setTransform()")}} が、入力パラメーターとして {{domxref("SVGMatrix")}} オブジェクトと同様に {{domxref("DOMMatrix")}} オブジェクトもサポートしました ({{bug(1565997)}})。

#### Media、WebRTC、Web Audio

- {{domxref("RTCStatsType")}} が `remote-outbound-rtp` である統計レコードのリモートタイムスタンプを、Firefox でサポートしました。これらの統計情報を提供するために使用する {{domxref("RTCRemoteOutboundRtpStreamStats")}} ディクショナリーに、{{domxref("RTCRemoteOutboundRtpStreamStats.remoteTimestamp", "remoteTimestamp")}} プロパティが含まれるようになりました。これは統計値が収集または生成されたときの、リモートピアのタイムスタンプを表します ({{bug(1615191)}})。

#### 廃止

- 偶然にもウェブに公開されていた複数の Gecko 内部のイベント (`DOMWindowClose` など) を、意図したとおり内部限定にしました ({{bug(1557407)}})。

### WebAssembly

- [WebAssembly の Bulk memory operations](/ja/docs/WebAssembly/Understanding_the_text_format#大規模メモリー操作) をサポートしました ({{bug(1528294)}})。
- [WebAssembly の Reference types](/ja/docs/WebAssembly/Understanding_the_text_format#参照型) をサポートしました ({{bug(1637884)}})。
- [WebAssembly の Threads](/ja/docs/WebAssembly/Understanding_the_text_format#webassembly_スレッド) (Shared memory および Atomics) をサポートしました ({{bug(1389458)}}, {{bug(1648685)}})。

## アドオン開発者向けの変更点

- 新しい API: [`tabs.warmup()`](/ja/docs/Mozilla/Add-ons/WebExtensions/API/tabs/warmup) ({{bug(1402256)}})
- [ストレージのクォータが、`sync` ストレージ領域に適用されるようになりました](/ja/docs/Mozilla/Add-ons/WebExtensions/API/storage/sync#Storage_quotas_for_sync_data) ({{bug(1634615)}}) ([addons.mozilla.org ブログの記事](https://blog.mozilla.org/addons/2020/07/09/changes-to-storage-sync-in-firefox-79/))

## 過去のバージョン

{{Firefox_for_developers(78)}}
