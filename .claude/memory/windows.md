# Windows の PC で動かすときの注意

embedded-debug-loop 0.3.0 で、利用者が「必要なpcはwindowsです。linux前提で作られてるなら見直して」と指摘した（2026-11-05）。生成プラグインのスキルも、利用者の PC で動くことを前提に作る。

## Copilot CLI のコマンド

- Copilot CLI は、Windows ではコマンドを PowerShell（6 以上）で動かす（Copilot CLI のコードと公式のインストールの文書からの読み取り。Windows の実機では確かめていない）。
- SKILL.md のコマンドは、bash でしか動かない書き方（ヒアドキュメント、`source`、`$(…)`）に頼らない。値を標準入力で渡すときは、PowerShell の here-string（`@'` と `'@`）をパイプで渡す書き方と、bash のヒアドキュメントの両方を書く。
- Python は、Windows では `py -3`（ない場合は `python`）、それ以外では `python3` で呼ぶ。SKILL.md の冒頭で `<python>` の呼び方を決め、コマンドはそれで書く。

## 文字コード

- スクリプトは、標準出力に UTF-8 のバイト列を直接書かない。日本語の Windows の PowerShell は、コンソールの文字コード（cp932 など）で読むため、文字化けする。
- Python 3.15（2026-10 に出た）からは UTF-8 モードが既定で、パイプへの出力が UTF-8 になる（PEP 686）。そのため、Windows でパイプに書くときは、コンソールの文字コード（`GetConsoleOutputCP`）に合わせる。書けない文字は JSON の `\uXXXX` にする。CI の Python は 3.8 と 3.12 で、3.15 での文字化けとその直しは確かめていない。日本語の Windows は、`PYTHONIOENCODING=cp932` などで真似ただけで、実機では確かめていない。
- Copilot CLI がコマンドの出力をパイプで受け取るのか、疑似コンソール（ConPTY）で受け取るのかは確かめていない。疑似コンソールなら Python はコンソールに直接書くので、パイプのときの直しが効く場面は少ない。
- 標準入力の値は、バイト列で読み、BOM を外し、UTF-8 で読めなければ OS の既定の文字コードで読む。最後の改行（`\r\n` か `\n`）を1つ外す。
- 設定ファイルとログは、いつも UTF-8 で書く。

## そのほか

- ビルドのシェルは、Windows では cmd か PowerShell を選べるようにした（SDK の環境設定ファイルは、cmd では `call`、PowerShell では `.` で読み込む）。Linux 用の SDK（Yocto など）は、WSL の中で使う。
- 時間切れで止めるときは、Windows では `taskkill /T /F` で子のプロセスまで止める。
- cmd は、コマンドが見つからないとき、`%ERRORLEVEL%` は 9009 でも、プロセスの終了コードは 1 になる（GitHub Actions の Windows で分かった）。
- PowerShell の `Invoke-Expression` の後の `$?` は、中の `cd` などが失敗しても真のままのことがある（GitHub Actions で分かった）。
- embedded-debug-loop のシリアルは、利用者の選択で、pyserial 3.5（BSD）をスキルのフォルダに同梱した（ライセンスの文も同梱）。テストは、pyserial の `socket://` で TCP の模擬のコンソールにつなぐので、Windows でも動く。
- Windows で動くかは、GitHub Actions の `windows-latest` でスキルのテストを動かして確かめた（公開リポジトリなので費用はかからない）。Windows の Copilot CLI からの使い方は、まだ確かめていない。
