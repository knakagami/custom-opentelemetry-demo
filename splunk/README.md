<!-- markdownlint-disable-next-line -->

# Welcome to the OpenTelemetry Astronomy Shop Demo

This repository contains a fork of the OpenTelemetry Astronomy Shop, a microservice-based
distributed system intended to illustrate the implementation of OpenTelemetry in
a near real-world environment. It includes customizations for use with Splunk
Observability Cloud.

## Update Docker and Kubernetes Scripts

After synchronizing changes with the upstream repository, the following
command can be used to update the Splunk versions of the docker-compose.yml
and kubernetes/opentelemetry-demo.yaml files, which are optimized for use
with Splunk Observability Cloud:

```bash
./update-demos.sh
```

## Quick start

You can be up and running with the demo in a few minutes. Check out the docs for
your preferred deployment method:

- [Docker](https://lantern.splunk.com/Data_Descriptors/Docker/Setting_up_the_OpenTelemetry_Demo_in_Docker)
- [Kubernetes](https://lantern.splunk.com/Data_Descriptors/Kubernetes/Setting_up_the_OpenTelemetry_Demo_in_Kubernetes)

## Local Customize

このカスタムデモレポジトリには、`product-catalog` サービスに対して新規デプロイを実施したことによりアプリケーションのエラー率が上昇するというシナリオをデモするスクリプトが含まれます。
操作方法は以下の通りです。

- 前提
    - jp0/Splunk-JP-Sandbox Orgを利用します
    - Dashboard: [Splunk] Service Health Dashboard が設定されています
    - Splunk版 Astronomy Shop が既にデプロイされています
    - Splunk Distro of OpenTelemetry CollectorやAPMエージェントが既に設定されており、Splunk Observability Cloud上でそのデータを参照できます
- 事前準備
    - `custom-opentelemetry-demo/splunk` ディレクトリに `env` ファイルを作成し、IngestトークンおよびGitリポジトリのURLを設定します。
    - Splunk Observability Cloud上でGlobal Data Linkを構成します。
        - `service.name` 属性に対して、`${GIT_REPO}/tree/main/src/{{properties.[service.name]}}` の形式でリンクを設定
        - `service.version` 属性に対して、`${GIT_REPO}/release/tag/{{properties.[service.version]}}` の形式でリンクを設定
    - Gitレポジトリ上で、架空のリリースを作成します。
        - `2.0.1` と `2.1.8` を作成
        - カスタムのバージョンに変更したい場合は、`opentelemetry-demo-with-catalogerror.yaml` の `product-catalog` サービスの `OTEL_RESOURCE_ATTRIBUTES` セクションの `service.version` をあわせて変更
- エラーを含むバージョンのリリース
    - 以下のコマンドを実行することで、`product-catalog` サービスのエラーを含むバージョンをデプロイします。
    ```bash
    custom-opentelemetry-demo/splunk/release-new-productcatalogservice.sh new custom-o11y-demo-jp0 product-catalog 2.1.8
    ```
    - デプロイを実行すると、`product-catalog` サービスのエラー率が上昇します。
    - ダッシュボード上には、アプリケーションデプロイを示すイベントが表示されるとともに、新バージョンのタグに基づくチャートが表示されるようになります
        - カスタムイベントを開くと、Git Releaseというリンクが表示され、Gitリポジトリの該当のリリースに直接アクセスできます
    - Service Map、Tag Spotlightなどで、`service.version` 属性によるエラー率等の変化や、Global Data Linkに基づくGitレポジトリへの移動が可能になります
- 元のバージョンへの戻し
    - 以下のコマンドを実行することで、`product-catalog` サービスの元のバージョンをデプロイします。
    ```bash
    custom-opentelemetry-demo/splunk/release-new-productcatalogservice.sh revert custom-o11y-demo-jp0 product-catalog 2.0.1
    ```
    - デプロイを実行すると、`product-catalog` サービスのエラー率が元に戻ります。
        - ダッシュボード上には、アプリケーションデプロイを示すイベントが表示されるとともに、旧バージョンのタグに基づくチャートが表示されるようになります
