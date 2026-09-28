# renovate-config

globis-org 共通の Renovate preset。

## Presets

| File               | Description                                                                 |
|:-------------------|:----------------------------------------------------------------------------|
| sre.json           | SREチームの利用推奨設定 (githubActions を内包)。全リポジトリで最初に extends する |
| githubActions.json | GitHub Actions の推奨設定 (digest pin / グルーピング / major 以外の automerge) |
| terraform.json     | Terraform の推奨設定 (更新ポリシーと PR の分け方)                              |
| atlantis.json      | Atlantis で実行する場合の設定 (terraform を内包)                               |
| hcp.json           | HCP Terraform で実行する場合の設定 (terraform を内包)                          |

## Usage

`sre` を先に、実行基盤に対応する preset (`atlantis` / `hcp`) を後に書く。

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "github>globis-org/renovate-config:sre",
    "github>globis-org/renovate-config:hcp"
  ],
  "enabledManagers": ["terraform", "github-actions"]
}
```

**extends の順序は意味を持つ**。後に書いた preset のスカラー値と packageRules が勝つため、
`prHourlyLimit` は `atlantis` / `hcp` 側の値 (1) が有効になる。

## terraform.json の構成

更新ポリシー (パス非依存) と PR の分け方 (パス依存) を別々の rule に分けている。

| 種類 | 対象 | 内容 |
|:--|:--|:--|
| ポリシー | `terraform` manager 全体 | `rangeStrategy: pin`、major 以外は automerge |
| グルーピング | `terraform/**` 配下のみ | `terraform/` 直下のディレクトリ単位で PR とブランチを分ける |

`.tf` を `terraform/` 配下に置いていないリポジトリもポリシーは受け取れる。
グルーピングだけはリポジトリ側で定義すること。

## automerge の扱い

| manager | automerge する | automerge しない |
|:--|:--|:--|
| github-actions | `minor` / `patch` / `pin` / `pinDigest` / `digest` | `major`、`replacement` (action のリネーム追従) |
| terraform | `minor` / `patch` / `pin` / `digest` | `major`、`replacement` |

atlantis / hcp のどちらでも同じポリシーになる。実際のマージは Atlantis の apply 結果や
HCP Terraform の `hcp-terraform/plan` check が通ることが前提。

実行基盤ごとにポリシーを分けたくなった場合は、terraform.json のポリシー rule を消して
atlantis.json / hcp.json 側にそれぞれ置くこと (preset の解決順で後勝ちになる)。

## preset から継承される値

リポジトリ側に同じ値を書いても問題ない (暗黙の継承を避けて明示する方針のため)。
ここでは preset 側の既定値がどれかだけを示す。

| 設定 | 継承元 | 値 |
|:--|:--|:--|
| `minimumReleaseAge` | sre.json | `7 days` |
| `rangeStrategy` | terraform.json (terraform manager) | `pin` |
| `prHourlyLimit` | atlantis.json / hcp.json | `1` |
| `timezone` / `reviewers` / `labels` / `schedule` | sre.json | — |

`prConcurrentLimit` はリポジトリごとの処理能力に合わせて決めるため preset では指定しない
(Renovate のデフォルトは 10)。`enabledManagers` とリポジトリ固有の packageRules も同様に
リポジトリ側で指定する。
