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

