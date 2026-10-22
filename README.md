# 書籍 「基礎から学ぶ組込み Rust」 Chapter 4-02 "L チカ" サンプルの 2026 年 Rust 開発環境への対応版

このリポジトリでは、書籍 「 [基礎から学ぶ組込み Rust][book_embedded_rust] 」 の
Section-02 に掲載されている ["L チカ" サンプル][github_wio_terminal_blink_rs] に
対し、 2026-10-21 時点の Rust 開発環境に合わせた修正を試みています。

具体的には、

- Ubuntu Linux 24.04
- Wio Terminal (D51R)
- Rust 1.99.1 (5f94df478 2026-08-27)

という開発環境で発生する、以下の 3 件の mismatched types エラーと、 1 件の
deprecation warning の問題に、 "Rust の edition や依存クレートのバージョンを上げる"
という方針で対応しています。(もし、ビルド時に "there are multiple different versions
of crate `heapless` dependency graph" というエラーが発生する問題を修正したいのであれ
ば、コミット https://github.com/noric/wio-terminal-blink-rs-alternative/commit/7785d5ec28e32f57ecce669b62e22d40ebfcc638
の、 Cargo.toml ファイルへの依存性の追加による修正の方がシンプルで、書籍の内容からの
乖離もありません。)

```
$ cargo hf2
warning: `/learning_embedded_rust/wio-terminal-blink-rs/.cargo/config` is deprecated in favor of `config.toml`
  |
  = help: if you need to support cargo 1.38 or earlier, you can symlink `config` to `config.toml`
warning: `/learning_embedded_rust/wio-terminal-blink-rs/.cargo/config` is deprecated in favor of `config.toml`
  |
  = help: if you need to support cargo 1.38 or earlier, you can symlink `config` to `config.toml`

# ... snip ...

error[E0308]: mismatched types
   --> /rust/cargo/registry/src/index.crates.io-1949cf8c6b5b557f/wio_terminal-0.3.0/src/wifi.rs:135:19
    |
135 |             ssid: ssid.into(),
    |                   ^^^^^^^^^^^ expected `String<64>`, found `String<UInt<UInt<UInt<_, B0>, B0>, B0>>`
    |
note: there are multiple different versions of crate `heapless` in the dependency graph

# ... snip ...

error[E0308]: mismatched types
   --> /rust/cargo/registry/src/index.crates.io-1949cf8c6b5b557f/wio_terminal-0.3.0/src/wifi.rs:136:23
    |
136 |             password: pw.into(),
    |                       ^^^^^^^^^ expected `String<64>`, found `String<UInt<UInt<UInt<_, B0>, B0>, B0>>`
    |

# ... snip ...

error[E0308]: mismatched types
   --> /rust/cargo/registry/src/index.crates.io-1949cf8c6b5b557f/wio_terminal-0.3.0/src/wifi.rs:196:18
    |
196 |         rpc.args(&mut tx_buff);
    |             ---- ^^^^^^^^^^^^ expected `&mut Vec<u8, 64>`, found `&mut Vec<u8, _>`
    |             |
    |             arguments to this method are incorrect
    |
note: there are multiple different versions of crate `heapless` in the dependency graph

# ... snip ...

For more information about this error, try `rustc --explain E0308`.
error: could not compile `wio_terminal` (lib) due to 3 previous errors
warning: build failed, waiting for other jobs to finish...
```


## 修正箇所

### .cargo/config の Deprecation Warning

cargo コマンドのバージョン 1.78 から、 cargo 設定ファイルの既定値が
`.cargo/config` から `.cargo/config.toml` に正式に変更されていたため、
`PROJECT_ROOT/.cargo/config` ファイルを "PROJECT_ROOT/.cargo/config.toml" に
リネームしました。 cargo 設定ファイルの内容自体の変更はありません。

- [Deprecate non-extension .cargo/config files. - CHANGELOG Cargo 1.78 (2024-05-02)][cargo_changelog_1_78]

### mismatched types エラー

このエラーは、 wio_terminal 0.3.0 クレートが内部で使用している seeed-erpc 0.1.3
クレートとの競合により、依存クレートのツリーの中に二つのバージョンの heapless
クレートが含まれるようになってしまった事が直接の原因だったようです。

```
$ cargo tree
wio-terminal-blink-rs v0.1.0 (learning_embedded_rust/wio-terminal-blink-rs)
├── panic-halt v0.2.0
└── wio_terminal v0.3.0
    ├── atsamd-hal v0.12.0
    │   ├── atsamd51p v0.9.0

    # ... snip ...

    ├── generic-array v0.14.9 (*)
    ├── heapless v0.5.6        # <= wio-terminal-0.3.0 が heapless-0.5.6 に依存
    │   ├── as-slice v0.1.5 (*)

    # ... snip ...

    ├── seeed-erpc v0.1.3
    │   ├── bbqueue v0.4.12 (*)
    │   ├── bitfield v0.13.2
    │   ├── bitflags v1.3.2
    │   ├── generic-array v0.14.9 (*)
    │   ├── heapless v0.8.0   # <= seeed-erpc-0.1.3 が heapless-0.8.0 に依存
```

**なお、このエラーは、 Cargo.toml ファイルの dependencies セクションに、
`seeed-erpc = "= 0.1.1"` という行を追加して、明示的バージョンを 0.1.1 に固定する
ことでも解消可能です。** 単に書籍のサンプルを動かすことが目的である場合は、この
方法の方がシンプルです。詳しくは、 https://github.com/noric/wio-terminal-blink-rs-alternative/commit/7785d5ec28e32f57ecce669b62e22d40ebfcc638
のコミットを参照して下さい。

このリポジトリでは、書籍の出版後の組込み Rust の変化を追うため、あえて wio_terminal
クレートのバージョンを 2026-10-21 時点で最新の `0.9.0` にアップグレードする方法を
採用しています。

wio_terminal 0.9.0 クレートでは、依存している atsamd_hal クレートで GPIO に
アクセスする為の API が変更されおり、この変更に合わせて src/main.rs の user_led
を取得する処理も変更しています。

- [atsamd_hal Module gpio - https://docs.rs/atsamd-hal/0.21.3][rs_docs_atsamd_hal_0_21_3_gpio]

ちなみに atsamd_hal クレートで行われた GPIO の API は、
`user_led.into_push_pull_output()` のような関数を呼び出す方式から、書籍の
Chapter 3-13 "型変換" で解説されている Into / From トレイトを利用した
型パラメータを指定する方式に変更になっているようです。

なお、このインターフェースは、上記の atsamd_hal クレートのドキュメントに記載
されているように、一般的には、

``` rust
let mut user_led: Pin<_, PushPullOutput> = pins.pa15.into();
```

のように `into()` 関数を使用する習慣があるようでしたが、このリポジトリの修正では、
既存コードとの見た目の一貫性を優先して、

``` rust
let mut user_led = Pin::<_, PushPullOutput>::from(pins.user_led);
```

という記法を採用しています。これは単純に `user_led` という変数にだけ型指定を
したくなかっただけで、特に意味はありません。

### Rust の edition と依存ライブラリの更新

上記の .cargo/config ファイルの Deprecation Warning の修正と mismatched types
エラーの修正で、当初の目的であった "L チカ" は動作するようになりましたが、折角
なので、 Cargo.toml の Rust の edition と、もう一つの依存ライブラリである
panic-halt クレートのバージョンを、どちらも 2026-10-21 時点の最新に更新しました。


## 注意事項

このリポジトリは、書籍 「基礎から学ぶ組込み Rust」 の一読者が、書籍の内容の学習
の為に勝手に作成したもので、書籍やそのサポートサイトとは無関係です。この
リポジトリでの修正箇所のコードは無保証で、その利用は自己責任となります。十分に
ご注意下さい。

> THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
> IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
> FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
> AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
> LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
> OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
> SOFTWARE.


## 参照資料

### オリジナル実装

- [tomoyuki-nakabayashi / wio-terminal-blink-rs (フォーク元) - GitHub][github_wio_terminal_blink_rs]
- [オリジナル README.md][original_readme_md]

### .cargo/config の Deprecation Warning

- [Deprecate non-extension .cargo/config files. - CHANGELOG Cargo 1.78 (2024-05-02)][cargo_changelog_1_78]

### mismatched types エラー

- [Crate wio_terminal 0.9.0 - https://docs.rs/wio_terminal/0.9.0/wio_terminal/][rs_docs_wio_terminal_0_9_0]
- [atsamd_hal Module gpio - https://docs.rs/atsamd-hal/0.21.3][rs_docs_atsamd_hal_0_21_3_gpio]
  - atsamd_hal クレートの新しい GPIO  API (v2) に関する情報は、上のリンク先に
    まとめられている

[github_wio_terminal_blink_rs]: https://github.com/tomoyuki-nakabayashi/wio-terminal-blink-rs
[original_readme_md]: README-original.md
[book_embedded_rust]: https://www.c-r.com/book/detail/1403
[cargo_changelog_1_78]: https://doc.rust-lang.org/nightly/cargo/CHANGELOG.html#cargo-178-2024-05-02
[rs_docs_wio_terminal_0_9_0]: https://docs.rs/wio_terminal/0.9.0/wio_terminal/
[rs_docs_atsamd_hal_0_21_3_gpio]: https://docs.rs/atsamd-hal/0.21.3/atsamd_hal/gpio/index.html
