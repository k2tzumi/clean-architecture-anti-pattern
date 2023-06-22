---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
# page transition
transition: slide-left
# use UnoCSS
css: unocss
fonts:
  # basically the text
  sans: 'Noto Sans JP'
  # use with `font-serif` css class from windicss
  serif: 'Noto Serif JP'
  # for code blocks, inline code, etc.
  mono: 'Noto Sans Mono'
---

# クリーンアーキテクチャ　アンチパターン

<!-- June 22, 2023.  -->
v0.0.1

<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    Press Space for next page <carbon:arrow-right class="inline"/>
  </span>
</div>

<div class="abs-br m-6 flex gap-2">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon:edit />
  </button>
  <a href="https://github.com/k2tzumi/clean-architecture-anti-pattern/blob/main/slides.md" target="_blank" alt="GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---
transition: fade-out
layout: two-cols-header
---

# 自己紹介

katzumiと申します  

以下のアカウントで活動しています  
:tada: [祝初採択！](https://fortee.jp/phpconfukuoka-2023/proposal/9af6e2bc-b64a-4287-baef-ee17ddd21560)  
  
[コレ](https://fortee.jp/phpconfukuoka-2023/proposal/1cb61189-e941-4881-83a3-72bce26039ae)の供養です


::left::

<img src="https://pbs.twimg.com/profile_images/799890486773170176/KN4gKfS2_400x400.jpg" class="rounded-full w-40 mt-16 mr-12"/>

<logos-twitter /> [katzchum](https://twitter.com/katzchum)

::right::

<img src="https://avatars.githubusercontent.com/u/1182787?v=4" class="rounded-full w-40 mt-16 mr-12"/>

<logos-github-octocat /> [k2tzumi](https://github.com/k2tzumi)  
<simple-icons-zenn /> [katzumi](https://zenn.dev/katzumi)  



<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>


---
transition: slide-up
layout: default
---

# よく見る図

![CleanArchitecture](https://blog.cleancoder.com/uncle-bob/images/2012-08-13-the-clean-architecture/CleanArchitecture.jpg)
[The Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)

---

# 話す内容

DDD,クリーンアーキテクチャなプロジェクトで？ :thinking_face: となった気づきです。　　

自分も含めてクリーンアーキテクチャの理解の解像度が低くてなってしまったのであるあるーという生暖かい気持ちで見ていただけると :pray:

---

# パターン１：ドメインモデルの貧血症

* 定義されているメソッドがコンストラクタとgetterのみ
* コンストラクタに制約もなし
* DTO?とモデルが区別ついてない？

---

# パターン１の問題点

* 多分他のレイヤーにロジックが散らばっている
* ドメインの制約がわからなくて関心事の認知負荷が大きい
* Valueオブジェクトに徹しようとし過ぎていている。。のか。。。？  
モデル間の変換処理を別の所でやっている？

---

# パターン1の処方

* まあちゃんとドメイン設計頑張ろうぜ
* コンストラクタで制約を表現する所からはじめよっか
* 制約をつける必要がない。。DTOなら [Array shapes](https://phpstan.org/writing-php-code/phpdoc-types#array-shapes)にしてドメインモデルにしなくても。。（要stan）  
```php {2}
/**
 * @phpstan-type UserAddress array{street: string, city: string, zip: string}
 */
class User
{
	/**
	 * @var UserAddress
	 */
	private array $address; // is of type array{street: string, city: string, zip: string}
}
```

---

# パターン２：　ReadオブジェクトとWriteオブジェクトを分けない

* テーブルの定義がそのままモデルのプロパティになっている
* データの永続化と読み込みの際のモデルが同じになっている

---

# パターン２の問題点

* テーブル設計 != モデル設計  
そもそもインフラ層の関心ごとをモデルに持ち込みたくないのに。。
* データベースで正規化してデータをうまく扱えない　　
書き込み時はコードで扱いコードに対する名称は扱わない  
逆に読み込み時は名称も表示したいケースがある　　
→ フロントエンドでコードに対する名称をN＋1なAPIアクセスするなど。。
* データの更新時に本来不要なデータが必要になる  
モデルが複雑で関連するデータが存在する場合（正規化されている場合もそう）に準備が必要になる
* 同タイミングにデータ更新された場合にデータが巻き戻ってしまう  
不必要なカラムまで更新しにいきがちで、最悪キー情報も更新してしまっていた例も

---

# パターン２の処方

* ReadオブジェクトとWriteオブジェクトを分ける  
コマンド・クエリ分離の原則（CQS）に従うこと
* レコードの区分値のみを更新する様な場合は事実（イベント）だけ保存できないか？も検討すること  
よくあるフラグのみを更新するような作りだと、更新対象以外のカラムにも影響が出やすい　　
独立したイベントとして扱いCQRSまで視野を入れたほうがいい


---

# パターン３：　ユースケース毎に適切なコマンドやクエリが作成されていない

* インフラ層でデータの新規登録と更新用のメソッドが１つ  
新規登録時にレコードを登録するコードもしくはIDが初番されてない
* なんでもできてしまう汎用的なコマンドが爆誕する　 
関連するテーブルが一つであればコマンドは一つという思い込み
* 全レコードを参照して集計するユースケースでも、一覧表示用のクエリを使い回している

---

# パターン３の問題点

* 新規と更新とで制約が違うのでモデルは分けたいのにできない  
→ コードが発行されていない状態をnullで表現とかするの？  
  そもそもコードの初番をどこで行うの？
* データのパターンが多いし、責務が多いのでテストしづらい
* オブジェクトが大量にロードされてメモリ不足や速度低下が発生する

---

# パターン３の処方

* 適切にコマン度を分けてコマンドに渡すドメインモデルにそれぞれの制約をつける
* 必要に応じて集計関数を利用したクエリを定義する

---

# パターン４：　集約を扱うモデルを適切に定義できてない

* 集約モデルの独自コレクションクラスがなくて毎回Where条件が違う処理をインフラ層に定義しまっている  
* 生のオブジェクト配列をユースケースでソート・フィルタリングしてしまっている
* 集約モデルの独自コレクションでメモリ上で大量データをフィルタリングしてしまっている

---

# パターン４の問題点

* クエリが増え過ぎてしまう  
ソート条件が変わるだけでもクエリ作るの辛い
* 汎用的に使えるクエリを定義できなくもないが、実装難しい
* オンメモリでのソートフィルタイングはパフォーマンス悪い
* 複数のユースケースに同じようなソートやフィルタリング処理がコピペ量産されてしまう  
ソースの可読性やテストが辛くなる

---

# パターン４の処方

＊ 集約のモデルをちゃんと定義してフィルターやソートができるようにする
* 逆にクエリに任せたほうが良い場合もあるので頑張り過ぎない  
DBの力を借りる  
ドメインロジックが漏れ出してしまっているという見え方もするかもだけれども

---

# その他パターン
賛否両論ありそうなパターン

* ドメイン層のロジックにフレームワーク依存のコードが混在  
→ フレームワークを変えたくなったらどうするの？
* 逆に原理主義に走りすぎてフレームワークの力を使えていない  
→　流石にコレクションクラスはどこのフレームワークにもあるでしょう
* トランザクションを全然使えていない（パターン３の延長線）  
関連するモデルを永続化する際に、汎用的なコマンドを複数組みわせてユースケースから呼び出しているケース  
→ 一見問題ないように見えるけれど、途中のコマンドで失敗した場合に辛い。要所要所ではトランザクションは使いたい
* 冪等性担保の配慮が全くされていない  
* イミュータブルなモデルになってない  

---
layout: end
---

ご清聴ありがとうございます