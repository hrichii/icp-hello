# イベント参加
2月22日（土）に竹芝で行われた[「ICPのサンプルキャニスター(スマコン)をデプロイする会 〜参加すればICPハッカソンWave3が提出できる！〜」](https://lu.ma/9obsceeh)に参加させていただきました！

参加者の方々とICPのメリットやデメリット、活用事例について詳しく討論させていただきました。

特にprincipalがサービス単位で払い出されることや、アンカーとパスキーを使った認証などはWave4のdApp開発のヒントになりました^^

機会を設けていただいた運営の方々、ありがとうございました！！

<img width=800px src="https://github.com/user-attachments/assets/33a2889f-3454-4303-9371-963068add755"/>

# サンプルキャニスターのデプロイ手順
## 1. 開発環境の構築
まずCommand Line Developer Toolsをインストールします。
公式ドキュメントにはRosetta2のインストールも推奨されていましたが、Apple M1 (MacOS 14.6.1)の私の環境ではRosetta2のインストール不要で動作しました。
```sh
$ xcode-select --install
```

## 2. miseのインストール
nodeやrustのインストールをする前に、ツールやランタイムの管理ツール`mise`をインストールします。
- [miseとは](https://mise.jdx.dev/about.html)
- [mise ではじめる開発環境構築](https://zenn.dev/takamura/articles/dev-started-with-mise)
```sh
$ brew install mise
$ echo 'eval "$(mise activate zsh)"' >> ~/.zshrc
$ source ~/.zshrc
```

## 3. Node.jsのインストール
Node.jsのインストールのため、以下コマンドを実行します。
```sh
$ mise use -g node@20
$ node --version
// バージョンが表示されればOK
```

## 4. Rustのインストール
Rustのインストールのため、以下コマンドを実行します。
```sh
$ mise use -g rust@1.85
$ ruby --version
// バージョンが表示されればOK
```

## 5. WebAssemblyモジュールを追加
CanisterはWebAssemblyモジュールを実行するため、Rustのビルドターゲットにwasm32-unknown-unknownを追加するため、以下コマンドを実行します。
```sh
$ rustup target add wasm32-unknown-unknown
```
以下確認コマンドで`wasm32-unknown-unknown`が追加されていればOK
```sh
$ rustup show
Default host: aarch64-apple-darwin
rustup home:  /Users/xxxx/.local/share/mise/installs/rust/1.85.0

installed targets for active toolchain
--------------------------------------

aarch64-apple-darwin
wasm32-unknown-unknown

active toolchain
----------------

1.85.0-aarch64-apple-darwin (default)
rustc 1.85.0 (4d91de4e4 2025-02-17)
```

## 6. IC SDKのインストール
まずは以下コマンドを実行し、ICSDKをインストールします。
```sh
$ sh -ci "$(curl -fsSL https://internetcomputer.org/install.sh)"
This path will then be added to your PATH environment variable by
modifying the profile files located at:

   /Users/xxxx/.profile
   /Users/xxxx/.zshenv

Current installation options:

            dfx version: latest
   modify PATH variable: yes

Proceed with installation?: Proceed with installation (default)
dfxvm is installed now.

To get started you may need to restart your current shell.
This would reload your PATH environment variable to include
the dfxvm bin directory.

To configure your shell, run:
  source "$HOME/Library/Application Support/org.dfinity.dfx/env"
```
上記コマンドの一番最後にもあるsourceコマンドを実行することでdfxコマンドが使用できるようになります。
```sh
$ source "$HOME/Library/Application Support/org.dfinity.dfx/env"
$ dfx --version
dfx 0.24.3
```

## 7. 開発者アイデンティティの作成
デプロイされると、ストレージやコンピューティングリソースが消費され、費用が発生します。
費用を払うために、開発者IDを作成する必要があります。以下コマンドでIDを作成します。
```sh
$ dfx identity new username
$ dfx identity use xxxx
$ dfx identity get-principal
xxxxx-xxxxx-xxxxx-xxxxx-xxxxx-xxxxx-xxxxx-xxxxx-xxxxx-xxxxx-xxx
```

## 8. プロジェクトの作成
helloという名前でプロジェクトを作成します。
今回は、BackendはRust、FrontendはSvelteKitを選択しました。
```sh
$ dfx new hello
```
```sh
? Select a backend language: ›  
  Motoko
❯ Rust
  Python (Kybra)
  Typescript (Azle)
```
```sh
? Select a frontend framework: ›  
❯ SvelteKit
  React
  Vue
  Vanilla JS
  No JS template
  None
```

## 9. LocalCanisterの起動
まずはローカルPC上で動作を確認するために、LocalCansterの環境を起動します。
以下のdfx startコマンドでを実行します。以下のコマンドであれば`Control+C`で止められますが、
--backgroundオプションを付与するとバックグラウンドで実行されるため、別途止めるためのコマンドが必要です。
```sh
$ cd hello
$ dfx start
```

## 10. Localへのデプロイ
以下のdfx deployコマンドを実行すると、ビルドが走り、Local Canister実行環境へdeployされます。
表示されたURLにアクセスすると以下画像のような画面が表示されます。

```sh
$ dfx deploy
Deployed canisters.
URLs:
  Frontend canister via browser:
    hello_frontend:
      - http://xxxxxx-cai.localhost:4943/ (Recommended)
      - http://127.0.0.1:4943/?canisterId=xxxxxx-cai (Legacy)
  Backend canister via Candid interface:
    hello_backend: http://127.0.0.1:4943/?canisterId=xxxxxx-cai&id=xxxxxx
```

## 11. Playgroundへのデプロイ
先ほどのコマンドに--playgroundを付与するだけでPlaygroundへデプロイが可能です。
Playgroundは費用が発生せず20分だけしか起動しないため、動作確認目的で使用すると良いでしょう。
以下が実行例です。
```sh
$  dfx deploy --playground
Deploying all canisters.
All canisters have already been created.
WARN: Cannot check for vulnerabilities in rust canisters because cargo-audit is not installed. Please run 'cargo install cargo-audit' so that vulnerabilities can be detected.
Executing: cargo build --target wasm32-unknown-unknown --release -p hello_backend --locked
    Finished `release` profile [optimized] target(s) in 0.13s
Installed code for canister hello_backend, with canister ID xtnc2-uaaaa-aaaab-qadaq-cai
WARN: This project uses the default security policy for all assets. While it is set up to work with many applications, it is recommended to further harden the policy to increase security against attacks like XSS.
WARN: To get started, have a look at 'dfx info canister-security-policy'. It shows the default security policy along with suggestions on how to improve it.
WARN: To disable the policy warning, define "disable_security_policy_warning": true in .ic-assets.json5.
Installed code for canister hello_frontend, with canister ID 5jw7w-wiaaa-aaaab-qacza-cai
Deployed canisters.
URLs:
  Frontend canister via browser:
    hello_frontend: https://5jw7w-wiaaa-aaaab-qacza-cai.icp0.io/
  Backend canister via Candid interface:
    hello_backend: https://a4gq6-oaaaa-aaaab-qaa4q-cai.raw.icp0.io/?id=xtnc2-uaaaa-aaaab-qadaq-cai
```
<img width="1439" alt="image" src="https://github.com/user-attachments/assets/b9784f34-db9a-4f7a-848f-a81e52745cf2" />

