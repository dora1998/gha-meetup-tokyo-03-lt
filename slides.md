---
theme: default
title: GitHub Actionsの痒いところを埋めるサードパーティーランナー
mdc: true
fonts:
  sans: BIZ UDPGothic
---

# GitHub Actionsの痒いところを埋める<br/>サードパーティーランナー

<div>
2024/5/16 GitHub Actions Meetup Tokyo #3

どら (@dora1998)

</div>

<div class="abs-br m-6 flex gap-2">
  <a href="https://github.com/dora1998/gha-meetup-tokyo-03-lt" target="_blank" alt="GitHub" title="Open in GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

---

# 自己紹介

<div class="flex flex-items-center gap-16 mt-20 px-6">

<div>

![](https://github.com/dora1998.png){style="width: 200px;" .border-rd-full}

</div>

<div>

## どら{.mb-4}

<div class="flex gap-4 mb-4">
<p><carbon-logo-github /> @dora1998</p>
<p><carbon-logo-x /> @d0ra1998</p>
</div>

- 株式会社サイバーエージェント
  - WINTICKET Web テックリードマネージャー
- CIの高速化が好き

</div>
</div>

---

# セルフホステッド ランナーとは

- 任意のマシンでGitHub ActionのJobを実行することができる
  - [https://docs.github.com/ja/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners](https://docs.github.com/ja/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners)
  - つまり、GitHubが提供していない環境も使える

---

# サードパーティーランナーとは

- セルフホステッド ランナーの仕組みを使ってランナーを提供するプロバイダーが存在する
  - まとめているリポジトリもあります
    - [https://github.com/neysofu/awesome-github-actions-runners](https://github.com/neysofu/awesome-github-actions-runners)
  - このブログ記事で知りました
    - [https://ikesyo.hatenablog.com/entry/github-actions-managed-runners](https://ikesyo.hatenablog.com/entry/github-actions-managed-runners)

---

# サードパーティーランナーの例

- Namespace
- BuildJet
- WarpBuild
- CodeBuild
- Cirun

---

# どういう時に使うといいか

- 料金を抑えたい
- ARM CPUやGPUを使いたい
- 実行時間を短くしたい
- クラウドストレージへアップロード/ダウンロードしたい
- より詳細なジョブの分析を行いたい

---

<style>
  table {
    --uno: text-sm;
  }
</style>

# 料金を抑えたい

プライベートリポジトリで使う場合、GitHub提供の半額程度になる

| サービス              | 2 vCPUで動かした場合           |
| --------------------- | ------------------------------ |
| GitHub (Private) [^1] | $0.008 (2 vCPU, 7GB, 14GB)     |
| Namespace [^2]        | $0.003 (2 vCPU, 4GB, 48GB[^3]) |
| BuildJet [^4]         | $0.004 (2 vCPU, 8GB)           |
| WarpBuild [^5]        | $0.004 (2 vCPU, 7GB, 150GB)    |
| CodeBuild [^6]        | $0.0024 (lambda.x86-64.2GB)    |

[^1]: https://docs.github.com/ja/billing/managing-billing-for-github-actions/about-billing-for-github-actions#per-minute-rates
[^2]: https://namespace.so/pricing (※月額に含まれるクレジット分は2/3)
[^3]: https://namespace.so/docs/architecture/containers#machine-resource-shapes
[^4]: https://buildjet.com/for-github-actions/docs/about/pricing
[^5]: https://docs.warpbuild.com/runners (※スポットインスタンスを使うとさらに25%安い)
[^6]: https://aws.amazon.com/jp/codebuild/pricing/

---

# ARM CPUやGPUを使いたい

- ARM64やGPUを搭載したランナーを使うことができる
- ちなみにこれらはGitHubでもベータで提供中
  - [https://github.com/github/roadmap/issues/961](https://github.com/github/roadmap/issues/961)
  - [https://github.com/github/roadmap/issues/836](https://github.com/github/roadmap/issues/836)

---

# 実行時間を短くしたい

- BuildJetやWarpBuildはシングルコア性能が高いことを強みとしている
  - [https://buildjet.com/for-github-actions/blog/a-performance-review-of-github-actions-the-cost-of-slow-hardware](https://buildjet.com/for-github-actions/blog/a-performance-review-of-github-actions-the-cost-of-slow-hardware)
  - [https://docs.warpbuild.com/#what-is-warpbuild](https://docs.warpbuild.com/#what-is-warpbuild)
  - [https://www.warpbuild.com/blog/github-actions-speeding-up#a-note-on-github-hosted-runner-processors](https://www.warpbuild.com/blog/github-actions-speeding-up#a-note-on-github-hosted-runner-processors)
- CPUヘビーなジョブであれば、同じコア数で実行時間が短くなるかも

---

# クラウドストレージへアップロード/ダウンロードしたい

- 例えば、S3を使う場合はAWS上のランナーを使えば高速にアクセスできる
- 実際のベンチマークでもダウンロードが10倍ほど速かった (1 MB, 1000個, 16スレッド)
  - GitHub-hosted Runner: 31.4 MB/s
  - AWS CodeBuild: 293.0 MB/s

---

# より詳細なジョブの分析を行いたい

- Namespaceは特に管理画面が充実している
- GitHub上では確認できない以下のメトリクスが確認できる
  - 実行中のCPU/メモリ使用率の推移
  - 日ごと・ワークフローごとの使用量

---

# 気をつけたいこと

- 起動時間がかかる
- Cache, Artifactが遅い
- ランナーの差異
- セキュリティ

---

## 気をつけたいこと

# 起動時間がかかる

- GH-hosted runnerよりもジョブの実行開始までラグがある
- GH-hosted ※ほぼゼロ > Namespace, BuildJet > CodeBuild >>> Cirun (GCP) ※1分程度

---

# Cache, Artifactが遅い

- actions/cache, actions/{download,upload}-artifact はGitHubのストレージとやり取りする
- GH-hosted runnerだと（おそらく内部で繋がってるので）速いが、セルフホステッドランナーだと遅くなる
- 各社置き換えを提供しているので、その使い勝手を見た方が良い

---

# ランナーの差異

- そのままランナーだけ書き換えると微妙に動かないことがある
  - パッケージが足りない
    - Namespaceのランナーではlessが標準ではないとか
  - CodeBuildではAWS操作時にLambdaに設定されたサービスロールで認証されている

---

# セキュリティ

- リポジトリ(WarpはOrgも)のAdmin権限を渡す必要がある
  - これはランナーを登録するために必要
    - [https://discord.com/channels/975088590705012777/975088591183171597/1168482784696873010](https://discord.com/channels/975088590705012777/975088591183171597/1168482784696873010)
  - 一方で、リポジトリの削除など重大な操作もできてしまうので、万が一管理体制に不備があると重大インシデントにつながりうる
    - [https://docs.github.com/en/rest/authentication/permissions-required-for-github-apps?apiVersion=2022-11-28#repository-permissions-for-administration](https://docs.github.com/en/rest/authentication/permissions-required-for-github-apps?apiVersion=2022-11-28#repository-permissions-for-administration)

---

# まとめ

- サードパーティランナーにはGitHub-hosted ランナーにはない特徴がある
- 使う上では…
  - 起動まで若干ラグがある
  - Cache, Artifactをそのまま使用すると遅いので対策が必要
  - ランナーの差異で調整が必要かも
  - GitHub Appが要求する権限が広いので注意
