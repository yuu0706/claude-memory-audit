# claude-memory-audit

| 項目 | 内容 |
| --- | --- |
| 対象 | Claude Codeのメモリ（各設定ディレクトリの`projects/*/memory`）の精査 |
| 動作環境 | 必須: macOS、Xcodeのコマンドラインツール（`/usr/bin/python3`3.9以上の実体）、Claude Code（`claude`）。任意: `jq`（作成したリポジトリを記録するフックに使う） |
| 前提のリポジトリ | 必須: [claude-change-review](https://github.com/yuu0706/claude-change-review)。任意: [claude-report-inbox](https://github.com/yuu0706/claude-report-inbox) |
| ライセンス | MIT（[LICENSE](LICENSE)） |

Claudeのメモリに書いたファイル・コマンド・設定・前提が今も正しいかを確かめるスキル`memory-audit`を配布する。古い記述や重なったメモリを見つけ、直したいことを[claude-change-review](https://github.com/yuu0706/claude-change-review)の記録に足す。週1回の定期実行はメモリを直接書き換えない。

## 1. このスキルでできること

| できること | 頼み方の例 |
| --- | --- |
| メモリに書いたファイル・ディレクトリ・スクリプトが実在するかを確かめる | 「メモリを精査して」 |
| メモリに書いたコマンドが手元にあるかを確かめる | 「メモリを精査して」 |
| 同じ事実を書いた別のメモリや、食い違うメモリを見つけ、まとめる案を作る | 「メモリを精査して」 |
| CLAUDE.md・スキルに移った内容をメモリにも残していないかを確かめる | 「メモリを精査して」 |
| `MEMORY.md`の索引を、実在するファイルと一致させる案を作る | 「メモリを精査して」 |
| 週1回の精査の結果を開き、記録に足した直したいことと確かめられなかったものを伝える | 「メモリの結果を見て」 |

## 2. 使いどころの例

| 頼み方 | Claudeが行うこと |
| --- | --- |
| 「メモリを精査して」 | 否認リストと訂正リストを読み、対象のメモリを全部読む。実物と違う記述は確かめた事実に直す案にし、重なったメモリは新しく詳しいほうへまとめる案にする |
| 「メモリの結果を見て」 | 週1回の精査の結果を開き、記録に足した直したいことと確かめられなかったものを中心に伝える。`直してよい`は会話で入れ、`要確認`は承認を受けてから入れる |
| 定期実行（毎週月曜10:30） | 対象のメモリを複製し、作業用の複製だけを直させる。差分は追加・修正・削除・統合ごとに記録へ足し、元のメモリは変えない |

## 3. 扱わないこと

| 扱わないもの | 代わりに使うもの |
| --- | --- |
| メモリの直接の書き換えと削除 | 記録に足し、[claude-change-review](https://github.com/yuu0706/claude-change-review)の手順で会話のClaudeが入れる |
| AWS・ネットワーク・本番環境への問い合わせ | 確かめずに、結果の「確かめられなかったもの」に書く |
| 確かめる範囲の外（各リポジトリ・クラウドストレージの同期フォルダ・`~/Documents`など） | 確かめずに、理由を「対象外の場所」として結果に書く |
| CLAUDE.md・スキル・設定ファイルの修正 | スキルは[claude-skill-audit](https://github.com/yuu0706/claude-skill-audit)が精査する |

確かめる範囲は、各設定ディレクトリ・`~/.agents`・`~/claude-reports`・`~/.local/state/claude-reports`の下に限る。同期フォルダなどを検索すると、macOSがアクセス許可の画面を出すためである。

## 4. 前提

| リポジトリ | 要るか | 理由 |
| --- | --- | --- |
| [claude-change-review](https://github.com/yuu0706/claude-change-review) | 必須 | 定期実行は、`change-review`の`scripts/link-scopes.sh`で書き換えを拒否する規則を作ってから起動する。無ければ起動しない。直したいことを記録に足す`new-change.sh`・`propose.sh`も`change-review`にある |
| [claude-report-inbox](https://github.com/yuu0706/claude-report-inbox) | 任意 | 精査の結果と失敗を、会話の開始時に知らせる |

先にclaude-change-reviewを入れてから、このリポジトリを入れる。

## 5. 構成

| パス | 内容 |
| --- | --- |
| `install.sh` | 入れる・外す。オプションは`--with-schedule`・`--no-schedule`・`--yes`・`--dry-run`・`--uninstall`。ほかのリポジトリと同じもので、`core/skills/`にあるスキルの分だけを扱う |
| `core/skills/memory-audit/SKILL.md` | 精査の範囲・手順・結果の構成 |
| `core/skills/memory-audit/scripts/run.sh` | 週1回の精査を起動する。対象を複製し、権限を絞った`claude -p`を動かし、差分を記録へ足す。`--dry-run`で組み立てたコマンドだけを表示する |
| `core/skills/memory-audit/scripts/targets.conf` | 対象のメモリのディレクトリ。既定は`$CONFIG_DIR/projects/*/memory`の1行 |
| `core/skills/memory-audit/scripts/tag-memory-repo.sh`・`backfill-memory-repo.sh` | メモリを作ったときのリポジトリを記録するフックと、既存のメモリへの埋め戻し（任意） |
| `core/skills/memory-audit/scripts/README.md` | 起動の内容・権限の規則・記録への追記の詳細 |
| `core/skills/memory-audit/launchd/memory-audit.plist.template` | launchdのplistのひな形。毎週月曜10:30に起動する |
| `core/tests/` | テスト。`core/tests/run.sh`で動く |
| `LICENSE` | MIT License |

結果は`~/claude-reports/memory-audit/<YYYY-MM-DD>.md`に書く。直す前の全メモリは`~/claude-reports/memory-audit/.snapshots/<日時>/`へ複製してから起動する。複製と作業用のフォルダ`.staging/`は自動では消さない。

## 6. 入れ方

1. 先に[claude-change-review](https://github.com/yuu0706/claude-change-review)を入れる。
2. リポジトリをcloneする。`~/Documents`・`~/Desktop`・`~/Downloads`の下には置かない。launchdから動く定期実行は、macOSのプライバシー保護により、これらのフォルダのファイルを開けないことがある。

   ```sh
   git clone https://github.com/yuu0706/claude-memory-audit ~/claude-tools/claude-memory-audit
   cd ~/claude-tools/claude-memory-audit
   ```

3. 行う操作を確かめてから入れる。途中で週1回の精査を登録するかを聞かれる。

   ```sh
   ./install.sh --dry-run
   ./install.sh
   ```

`install.sh`は次を行う。何回実行しても結果は同じになる。

| No | 操作 | 備考 |
| --- | --- | --- |
| 1 | 必要なもの（Xcodeのコマンドラインツール・Python 3.9以上・`claude`）を確かめる | 足りなければ入れる前に`[y/N]`で聞く。`--yes`を付けると聞かずに進む。`claude`が無ければ定期実行を登録しない |
| 2 | `core/skills/memory-audit`を、スキル置き場（各設定ディレクトリの`skills`）へシンボリックリンクで置く | 同じ名前のものが既にあれば`<スキル置き場>/.trash/<YYYYMMDD-HHMMSS>-<名前>`へ移してから置く |
| 3 | 状態の置き場所`~/.local/state/claude-review-inbox/`を作る | 登録した定期実行の台帳`scheduled-jobs.tsv`を置く |
| 4 | 週1回の精査を、聞いてからlaunchdに登録する | plistは`~/Library/LaunchAgents/com.<ユーザー名>.memory-audit.plist`。同じスクリプトを指すplistが既にあれば、そのラベルを引き継ぐ |

### 6.1 週1回の精査

定期実行は週1回`claude -p`を動かすため、利用者のClaudeの利用枠を使う。そのため、`install.sh`は登録する前に聞く。

| 実行のしかた | 精査の扱い |
| --- | --- |
| `./install.sh`（端末から実行） | `週1回の見直し（memory-audit）を登録しますか。claude の利用枠を使います [y/N]`と聞く。`y`なら登録し、それ以外なら登録しない。登録済みなら外す |
| `./install.sh --with-schedule` | 聞かずに登録する |
| `./install.sh --no-schedule` | 聞かずに登録しない。登録済みなら外す |
| 標準入力が端末でなく、上の2つの指定も無い | 登録を変えない |
| `claude`が見つからない | 聞かずに登録しない |

### 6.2 環境ごとの設定

共通の設定ファイル`${XDG_CONFIG_HOME:-~/.config}/claude-review-inbox/jobs.conf`に、`名前=値`の行で書く。同じ名前の環境変数があれば、そちらを使う。`install.sh`はこのファイルを作らない。

| 項目 | 既定値 | 用途 |
| --- | --- | --- |
| `CLAUDE_LAUNCHER` | `claude` | 定期実行が起動するコマンド。語のうち`{job}`はジョブ名に置き換える |
| `CLAUDE_REVIEW_CONFIG_DIRS` | `~/.claude` | Claudeの設定ディレクトリの一覧（コロン区切り）。スキルのリンクを置く場所と、精査の対象がここから決まる |
| `CLAUDE_REVIEW_DENY_READ` | なし | 定期実行に読ませない場所（コロン区切り）。`~/Library/CloudStorage`は常に拒否する |

次は環境変数で指定する。

| 変数 | 既定値 | 用途 |
| --- | --- | --- |
| `MEMORY_AUDIT_TARGETS` | `scripts/targets.conf` | 対象を書いた設定ファイル |
| `CLAUDE_REPORTS_ROOT` | `~/claude-reports` | 結果・複製・作業用のフォルダの置き場所 |
| `CLAUDE_REPORTS_STATE` | `~/.local/state/claude-reports` | ロックとログの置き場所 |
| `CHANGE_REVIEW_SCRIPTS` | なし | `change-review`の`scripts`の場所。無ければスキル置き場の`change-review/scripts`を探す |

利用者の環境だけの決まりは、上書きファイル`${XDG_CONFIG_HOME:-~/.config}/claude-skills/memory-audit.local.md`に書く。あれば最初に読み、本文と食い違う箇所は上書きファイルに従う。

メモリを作ったときのリポジトリを記録するフック`tag-memory-repo.sh`は、`install.sh`では登録しない。使う場合は、利用者がsettings.jsonのPostToolUseに登録する。既存のメモリは`backfill-memory-repo.sh --apply`で埋め戻せる。リポジトリを置いているフォルダは`MEMORY_AUDIT_GIT_ROOT`で指定する。

## 7. 更新

```sh
cd ~/claude-tools/claude-memory-audit
git pull
./install.sh
```

スキルはリポジトリの中のディレクトリへのシンボリックリンクのため、`git pull`だけで新しい版になる。plistのひな形が変わったときは`./install.sh --with-schedule`で登録し直す。

## 8. 外し方

```sh
./install.sh --uninstall --dry-run   # 行う操作を確かめる
./install.sh --uninstall
```

`install.sh`が登録した定期実行と、このリポジトリを指すスキルのリンクだけを消す。`.trash`へ移したものは自動では戻さず、戻すコマンドを表示する。結果・複製・直したいことの記録は消さない。

## 9. テスト

```sh
core/tests/run.sh
```

頭の`# requires:`に書いたスキルが無いテストは飛ばす。このリポジトリだけで動くのは、作成したリポジトリの記録のテスト（`scripts/test-tag.sh`）である。`run.sh`を確かめる`scripts/test.sh`は`change-review`を要するため、このリポジトリだけでは飛ばす。テストは本物の`HOME`・`launchctl`・`claude`に触れない。

## 10. 関係するスキル

| スキル | 関係 |
| --- | --- |
| [claude-change-review](https://github.com/yuu0706/claude-change-review) | 直したいことの記録と会話での反映を受け持つ。必須 |
| [claude-report-inbox](https://github.com/yuu0706/claude-report-inbox) | 精査の結果と失敗を、会話の開始時に知らせる |
| [claude-skill-audit](https://github.com/yuu0706/claude-skill-audit) | スキル群の精査を受け持つ |

## 11. ライセンス

MIT Licenseである。全文は[LICENSE](LICENSE)にある。
