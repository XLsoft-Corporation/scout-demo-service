# Docker Scout デモ

本リポジトリをクローンして、[Quickstart \| Docker Docs](https://docs.docker.com/scout/quickstart/) のドキュメントの手順を行ってみます。

## branch:v1

ビルドしてプッシュします。

```sh
docker build --push -t xlsoftpartner/docker-scout-demo:v1 .
```

Scout を Org に関連付けします。

```sh
docker scout enroll xlsoftpartner
```

リポジトリで Scout をオンにします。

```sh
docker scout repo enable --org xlsoftpartner xlsoftpartner/docker-scout-demo
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



