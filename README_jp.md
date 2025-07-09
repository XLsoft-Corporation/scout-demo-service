# Docker Scout デモ

本リポジトリをクローンして、[Quickstart \| Docker Docs](https://docs.docker.com/scout/quickstart/) のドキュメントの手順を行ってみます。

## branch:v1

ビルドしてプッシュします。

```sh
docker build --push -t xlsoftpartner/scout-demo:v1 .
```

Scout を Org に関連付けします。

```sh
docker scout enroll xlsoftpartner
```

リポジトリで Scout をオンにします。

```sh
docker scout repo enable --org xlsoftpartner xlsoftpartner/scout-demo
```

現時点での脆弱性をチェックしてみます。

```sh
$ docker scout quickview
    i New version 1.15.1 available (installed version is 1.15.0) at https://github.com/docker/scout-cli
    ✓ Image stored for indexing
    ✓ Indexed 79 packages
    ✓ Provenance obtained from attestation

    i Base image was auto-detected. To get more accurate results, build images with max-mode provenance attestations.
      Review docs.docker.com ↗ for more information.

  Target               │  local://xlsoftpartner/docker-scout-demo:v1  │    2C    20H     8M     4L     1?
    digest             │  70147fe1fa25                                │
  Base image           │  alpine:3                                    │    2C    15H     7M     0L     1?
  Refreshed base image │  alpine:3                                    │    0C     0H     1M     0L
                       │                                              │    -2    -15     -6            -1
  Updated base image   │  alpine:3.19                                 │    0C     0H     1M     0L
                       │                                              │    -2    -15     -6            -1
```

ベースイメージに脆弱性があることが分かります。次のブランチでベースイメージを更新してみます。


## branch:v2

[Dockerfile](./Dockerfile) を編集します。

2024/11/29 時点の `alpine:3.19@sha256:7a85bf5dc56c949be827f84f9185161265c58f589bb8b2a6b6bb6d3076c1be21` にアップデートします。

v2 としてビルドしてプッシュします。

```sh
docker build --push -t xlsoftpartner/scout-demo:v2 .
```

再度 quickview してみます。

```sh
docker scout quickview
    i New version 1.15.1 available (installed version is 1.15.0) at https://github.com/docker/scout-cli
    ✓ Image stored for indexing
    ✓ Indexed 86 packages
    ✓ Provenance obtained from attestation

    i Base image was auto-detected. To get more accurate results, build images with max-mode provenance attestations.
      Review docs.docker.com ↗ for more information.

  Target             │  local://xlsoftpartner/scout-demo:v2  │    0C     4H     2M     4L 
    digest           │  836fc6257f90                         │
  Base image         │  alpine:3.19                          │    0C     0H     1M     0L 
  Updated base image │  alpine:3.20                          │    0C     0H     1M     0L
                     │                                       │

What's next:
    View vulnerabilities → docker scout cves local://xlsoftpartner/scout-demo:v2
    View base image update recommendations → docker scout recommendations local://xlsoftpartner/scout-demo:v2
    Include policy results in your quickview by supplying an organization → docker scout quickview local://xlsoftpartner/scout-demo:v2 --org <organization>
```

かなり少なくなりました。

お勧め情報に基づいて `cves` オプションで再スキャンしてみます。

```sh
## Packages and Vulnerabilities

   0C     1H     1M     1L  express 4.17.1
pkg:npm/express@4.17.1

    ✗ HIGH CVE-2022-24999 [OWASP Top Ten 2017 Category A9 - Using Components with Known Vulnerabilities]
      https://scout.docker.com/v/CVE-2022-24999
      Affected range : <4.17.3
      Fixed version  : 4.17.3
      CVSS Score     : 7.5
      CVSS Vector    : CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:N/A:H

    ✗ MEDIUM CVE-2024-29041 [Improper Validation of Syntactic Correctness of Input]
      https://scout.docker.com/v/CVE-2024-29041
      Affected range : <4.19.2
      Fixed version  : 4.19.2
      CVSS Score     : 6.1
      CVSS Vector    : CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N

    ✗ LOW CVE-2024-43796 [Improper Neutralization of Input During Web Page Generation ('Cross-site Scripting')]
      https://scout.docker.com/v/CVE-2024-43796
      Affected range : <4.20.0
      Fixed version  : 4.20.0
      CVSS Score     : 2.3
      CVSS Vector    : CVSS:4.0/AV:N/AC:L/AT:P/PR:N/UI:P/VC:N/VI:N/VA:N/SC:L/SI:L/SA:L
```

node アプリで利用している `express` に脆弱性があるようです。

次のブランチで express をアップデートしましょう。


## branch:v3

`Fixed version  : 4.20.0` に基づいて [package.json](./package.json) を修正し、再度ビルドしてみます。

```sh
docker build --push -t xlsoftpartner/scout-demo:v3 .
```

同様に quickview します。

```sh
docker scout quickview
    i New version 1.15.1 available (installed version is 1.15.0) at https://github.com/docker/scout-cli
    ✓ Image stored for indexing
    ✓ Indexed 103 packages
    ✓ Provenance obtained from attestation

    i Base image was auto-detected. To get more accurate results, build images with max-mode provenance attestations.
      Review docs.docker.com ↗ for more information.

  Target             │  local://xlsoftpartner/scout-demo:v3  │    0C     0H     1M     2L 
    digest           │  a0020d2b8173                         │
  Base image         │  alpine:3.19                          │    0C     0H     1M     0L 
  Updated base image │  alpine:3.20                          │    0C     0H     1M     0L
                     │                                       │
```

これで元ドキュメントの Step 4 までが終わりました。次のブランチで Step 5 のポリシーの設定を行いましょう。

## branch:v4

Policy は Organization に基づくため、関連付けます。

```sh
docker scout config organization xlsoftpartner
```

関連付いた状態で再度 quickview してみます。

```sh
docker scout quickview

Policy status  FAILED  (3/7 policies met, 2 missing data)

  Status │                   Policy                    │           Results
─────────┼─────────────────────────────────────────────┼──────────────────────────────
  !      │ No default non-root user found              │
  ✓      │ No AGPL v3 licenses                         │    0 packages
  ✓      │ No fixable critical or high vulnerabilities │    0C     0H     0M     0L
  ✓      │ No high-profile vulnerabilities             │    0C     0H     0M     0L
  ?      │ No outdated base images                     │    No data
         │                                             │    Learn more ↗
  ?      │ No unapproved base images                   │    No data
  !      │ Missing supply chain attestation(s)         │    2 deviations
```

まずは root ユーザー以外で実行するように [Dockerfile](./Dockerfile) を修正します。

一番最後に `USER appuser` を追加します。

次に `--provenance=true` と `--sbom=true` を追加することで、メタデータと SBOM を付与した状態でビルド、プッシュします。

> メタデータを付与するには、containerd でビルドする必要があります。
> Docker Desktop の「Settings＞General＞Use containerd for pulling and storing images」にチェックを付けておいてください。

```sh
docker build --provenance=true --sbom=true --push -t xlsoftpartner/scout-demo:v4 .
```

quickview してみます。

```sh
docker scout quickview

Policy status  SUCCESS  (7/7 policies met)

  Status │                   Policy                    │           Results
─────────┼─────────────────────────────────────────────┼──────────────────────────────
  ✓      │ Default non-root user                       │
  ✓      │ No AGPL v3 licenses                         │    0 packages
  ✓      │ No fixable critical or high vulnerabilities │    0C     0H     0M     0L
  ✓      │ No high-profile vulnerabilities             │    0C     0H     0M     0L
  ✓      │ No outdated base images                     │
  ✓      │ No unapproved base images                   │    0 deviations
  ✓      │ Supply chain attestations                   │    0 deviations
```

おめでとうございます！無事ポリシーにも準拠したイメージを作成できました。


## branch:v5

Dockerfile の `FROM` が Digit 固定なので、2025/4/9 時点での最新版 3.21 にアップデートし、`package.json` も最新にしてみます。

まずは PUSH しないでローカルで確認します。

```sh
docker build --provenance=true --sbom=true -t xlsoftpartner/scout-demo:v5 .
```

Quickview してみます。

```sh
docker scout quickview

Policy status  FAILED  (6/7 policies met)

  Status │                     Policy                     │           Results
─────────┼────────────────────────────────────────────────┼──────────────────────────────
  ✓      │ Default non-root user                          │
  ✓      │ No AGPL v3 licenses                            │    0 packages
  !      │ Fixable critical or high vulnerabilities found │    0C     1H     0M     0L
  ✓      │ No high-profile vulnerabilities                │    0C     0H     0M     0L
  ✓      │ No outdated base images                        │
  ✓      │ No unapproved base images                      │    0 deviations
  ✓      │ Supply chain attestations                      │    0 deviations
```

vulnerabilities が見つかってしまいました。

```sh
## Packages and Vulnerabilities

   0C     1H     0M     0L  path-to-regexp 0.1.10
pkg:npm/path-to-regexp@0.1.10

Dockerfile (14:17)
RUN  apk add --no-cache npm \
 && npm i --no-optional \
 && npm cache clean --force \
 && apk del npm

    ✗ HIGH CVE-2024-52798 [Inefficient Regular Expression Complexity]
      https://scout.docker.com/v/CVE-2024-52798
      Affected range : <0.1.12
      Fixed version  : 0.1.12
      CVSS Score     : 7.7
      CVSS Vector    : CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:N/VI:N/VA:H/SC:N/SI:N/SA:N/E:P
```

サジェスチョンにもあるように、0.1.12 以上にアップデートする必要がありそうです。[path-to-regexp](https://www.npmjs.com/package/path-to-regexp?activeTab=versions) や [path-to-regexp search results](https://github.com/search?q=repo%3Aexpressjs%2Fexpress%20Path-to-RegExp&type=code) のページにも記載がありますが、express に含まれるライブラリです。

express の最新バージョンは 4.21.2 ですので、`package.json` のバージョンを `^4.20.0` に変更します。

```sh
docker build --provenance=true --sbom=true -t xlsoftpartner/scout-demo:v5 .
```

再度 `quickview` します。

```sh
docker scout quickview

docker scout quickview
    ✓ SBOM of image already cached, 98 packages indexed
    ✓ Policy evaluation completed

  Target     │  local://xlsoftpartner/scout-demo:v5  │    0C     0H     0M     0L 
    digest   │  0ff303433a22                         │
  Base image │  alpine:3.21                          │    0C     0H     0M     0L 

Policy status  SUCCESS  (7/7 policies met)

  Status │                   Policy                    │           Results
─────────┼─────────────────────────────────────────────┼──────────────────────────────
  ✓      │ Default non-root user                       │
  ✓      │ No AGPL v3 licenses                         │    0 packages
  ✓      │ No fixable critical or high vulnerabilities │    0C     0H     0M     0L
  ✓      │ No high-profile vulnerabilities             │    0C     0H     0M     0L
  ✓      │ No outdated base images                     │
  ✓      │ No unapproved base images                   │    0 deviations
  ✓      │ Supply chain attestations                   │    0 deviations
```

無事脆弱性が無くなりましたので、ビルド・プッシュします。

```sh
docker build --provenance=true --sbom=true --push -t xlsoftpartner/scout-demo:v5 .
```

> 備考:  
> 2025/7/9 に再ビルド・再プッシュしています。
