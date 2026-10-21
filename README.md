# 書籍 「基礎から学ぶ組込み Rust」 Chapter 4-02 "L チカ" サンプルの 2026 年 Rust 開発環境への対応版

このリポジトリでは、書籍 「 [基礎から学ぶ組込み Rust][book_embedded_rust] 」 の
Section-02 に掲載されている ["L チカ" サンプル][github_wio_terminal_blink_rs] に
対し、 2026-10-21 時点の Rust 開発環境に合わせた修正を試みています。

具体的には、

- Ubuntu Linux 24.04
- Wio Terminal (D51R)
- Rust 1.99.1 (5f94df478 2026-08-27)

という開発環境で発生する、以下の 3 件の mismatched types エラーと、 1 件の
deprecation warning の問題に対応しています。

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


## 参照資料

### オリジナル実装

- [tomoyuki-nakabayashi / wio-terminal-blink-rs (フォーク元) - GitHub][github_wio_terminal_blink_rs]
- [オリジナル README.md][original_readme_md]

[github_wio_terminal_blink_rs]: https://github.com/tomoyuki-nakabayashi/wio-terminal-blink-rs
[original_readme_md]: README-original.md
[book_embedded_rust]: https://www.c-r.com/book/detail/1403
