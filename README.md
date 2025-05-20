# socketio-rust-emitter

[![build status](https://github.com/epli2/socketio-rust-emitter/actions/workflows/ci.yaml/badge.svg?branch=master&event=push)](https://github.com/epli2/socketio-rust-emitter/actions)
[![socketio-rust-emitter at crates.io](https://img.shields.io/crates/v/socketio-rust-emitter.svg)](https://crates.io/crates/socketio-rust-emitter)
[![socketio-rust-emitter at docs.rs](https://docs.rs/socketio-rust-emitter/badge.svg)](https://docs.rs/socketio-rust-emitter)

A Rust implementation of [socket.io-emitter](https://github.com/socketio/socket.io-emitter).

## How to use

```rust
use chrono::Utc;
use std::thread;
use std::time::Duration;

let io = Emitter::new("127.0.0.1");
let _ = thread::spawn(move || loop {
    thread::sleep(Duration::from_millis(5000));
    io.clone().emit(vec!["time", &format!("{}", Utc::now())]);
}).join();
```

```rust
// Different constructor options.

//1. Initialize with host:port string
let io = Emitter::new("localhost:6379")
// 2. Initlize with host, port object.
let io = Emitter::new(EmitterOpts {
    host: "localhost".to_owned(),
    port: 6379,
    ..Default::default()
});
```

## Examples

```rust
let io = Emitter::new(EmitterOpts { host: "127.0.0.1".to_owned(), port: 6379, ..Default::default() });

// sending to all clients
io.clone().emit(vec!["broadcast", /* ... */]);

// sending to all clients in "game" room
io.clone().to("game").emit(vec!["new-game", /* ... */]);

// sending to individual socketid (private message)
io.clone().to(<socketid>).emit(vec!["private", /* ... */]);

let nsp = io.clone().of("/admin");

// sending to all clients in "admin" namespace
nsp.clone().emit(vec!["namespace", /* ... */]);

// sending to all clients in "admin" namespace and in "notifications" room
nsp.clone().to("notifications").emit(vec!["namespace", /* ... */]);
```

## 新規参加者向けドキュメント

以下は、本リポジトリに初めて参加する方へ向けた概要です。

### コードベースの概要

- ルートには `Cargo.toml`、`README.md`、`src/lib.rs` などがあり、Rust のクレートとして構成されています。
- メインの実装は `src/lib.rs` にまとまっており、Redis へメッセージを送信する `Emitter` 構造体や、設定用の `EmitterOpts` などが定義されています。
- `README.md` には `Emitter` の使い方や複数の初期化方法、ルーム・ネームスペース宛の送信例が記載されています。
- CI では GitHub Actions が用意されており、`cargo build`・`cargo test`・`cargo fmt`・`cargo clippy` などを実行します。
- テストでは `testcontainers` クレートを利用し、Docker 上で Redis を起動して検証しています。

### 知っておくべきポイント

1. **`Emitter` の基本的な操作**
   - `Emitter::new` で Redis 接続を初期化できます。接続方法は文字列、`EmitterOpts`、`redis::Client` のいずれからも生成可能です。
   - `.to("room")` でルームを指定し、`.of("/namespace")` でネームスペースを指定できます。
   - `.emit(vec![...])` を呼び出すとメッセージが Redis 経由で配信され、送信後は内部状態（ルーム・フラグ）がクリアされます。
2. **メッセージ形式**
   - 送信データは `rmp-serde` を利用した MessagePack 形式でエンコードし、Redis の `PUBLISH` で送信しています。
3. **テスト実行時の注意**
   - `cargo test` は Docker を利用して Redis を立ち上げるため、Docker 環境が必要です（ネットワークから Docker イメージを取得できる状態で実行します）。

### 次に学ぶと良いこと

- `src/lib.rs` の実装を読み、`Emitter` がどのようにメッセージを構築・送信しているか確認する。
- README のサンプルコードを実際に動かしてみて、ルームやネームスペースの挙動を体験する。
- GitHub Actions の設定を眺め、CI の流れや `cargo fmt`・`clippy` の基準を把握する。
- 必要に応じて Rust の `serde`、`redis` クレートの使い方を学んでおくと、コードの理解が深まります。

