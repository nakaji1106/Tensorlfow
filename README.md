# Tensorlfow Dockerfile for ppc64le

以下のDockerhubにおけるイメージのDockerfileを公開しております。
ppc64leアーキテクチャにおけるDockerコンテナ作成時にご活用ください。

https://hub.docker.com/r/nakaji1106/tensorflow-python3/

※上記のDockerHubは現在、修正中のため、こちらのGitHubのDockerfileにてビルドください。

```
# ls
Dockerfile
# docker build -t <Account name>/<Image name>:<Tag> .
```

上記手順にてビルドされたDockerImageには、以下のコンポーネントが含まれています。
コンテナ起動時の自動起動サービスとしては設定していないため、稼働後にサービスの立ち上げが必要となります

- Openssh server
- Jupyter Notebook

以上
