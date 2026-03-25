# Profiling Ledger Server with async-profiler

## Prerequisites

Ledger サーバーのセットアップが完了し、コンテナが起動できる状態であること。
セットアップ手順は [ledger-only/README.md](ledger-only/README.md) を参照。

## Setup

ホストのカーネルパラメータを設定する。非 root プロセスから `perf_events` でカーネルコールスタックを取得するために必要。

```shell
sudo sysctl kernel.perf_event_paranoid=1
sudo sysctl kernel.kptr_restrict=0
```

async-profiler をダウンロードして展開する。

```shell
APROF_VERSION=4.0
curl -OL https://github.com/async-profiler/async-profiler/releases/download/v${APROF_VERSION}/async-profiler-${APROF_VERSION}-linux-x64.tar.gz
tar xzf async-profiler-${APROF_VERSION}-linux-x64.tar.gz
```

docker-compose ファイルの `scalardl-ledger` サービスに、async-profiler のマウントとセキュリティオプションを追加する。

```shell
vim postgres/docker-compose-ledger.yml
```

以下の変更を `scalardl-ledger` サービスに適用する。

```yaml
  scalardl-ledger:
    # ... 既存の設定 ...
    volumes:
      - ./ledger.properties:/scalar/ledger/ledger.properties.tmpl
      - ../fixture/ledger-key.pem:/scalar/ledger-key.pem
      - ../fixture/trial-license-cert.pem:/scalar/license-cert.pem
      - ../async-profiler-4.0-linux-x64:/async-profiler
    # 追加: JAVA_OPTS にプロファイリング用フラグを追加
    environment:
      JAVA_OPTS: >-
        -Dlog4j.configurationFile=file:log4j2.properties
        -XX:+UnlockDiagnosticVMOptions
        -XX:+DebugNonSafepoints
    security_opt:                                                            # 追加
      - seccomp:unconfined                                                   # 追加
    cap_add:                                                                 # 追加
      - SYS_ADMIN                                                            # 追加
```

- `/path/to/async-profiler-${APROF_VERSION}-linux-x64` は実際の展開先パスに置き換えること。
- `-XX:+UnlockDiagnosticVMOptions -XX:+DebugNonSafepoints` は、async-profiler を後からアタッチする場合にプロファイリング精度を向上させるためのフラグ（[参考](https://github.com/async-profiler/async-profiler/blob/master/docs/GettingStarted.md)）。
- docker-compose の `environment` で `JAVA_OPTS` を指定すると Dockerfile の `ENV JAVA_OPTS` が上書きされるため、元の `-Dlog4j.configurationFile=file:log4j2.properties` も含めて記述すること。`JAVA_OPT_MAX_RAM_PERCENTAGE` は別の環境変数なので影響しない。

## Run the profiler

コンテナを起動する。

```shell
docker-compose -f postgres/docker-compose-ledger.yml up -d
```

Ledger コンテナ内の Java プロセスの PID を確認し、利用可能なプロファイリングイベントを表示する。

```shell
docker exec scalardl-samples-scalardl-ledger-1 /async-profiler/bin/asprof list jps
```

ベンチマーク実行中に CPU プロファイルを取得する（例: 60 秒間）。

```shell
docker exec scalardl-samples-scalardl-ledger-1 /async-profiler/bin/asprof -d 60 -f /tmp/flamegraph.html jps
```

> コンテナ内の PID `1` は dockerize（Go プロセス）であり Java プロセスではない。PID `1` を指定するとコンテナがクラッシュするため、必ず `jps` を使用して Java プロセスを自動検出すること。

結果をホストにコピーする。

```shell
docker cp scalardl-samples-scalardl-ledger-1:/tmp/flamegraph.html ./flamegraph.html
```

## Profiling options

### CPU profiling (default)

```shell
docker exec scalardl-samples-scalardl-ledger-1 /async-profiler/bin/asprof -d 60 -f /tmp/cpu.html jps
```

### Allocation profiling

```shell
docker exec scalardl-samples-scalardl-ledger-1 /async-profiler/bin/asprof -d 60 -e alloc -f /tmp/alloc.html jps
```

### Lock contention profiling

```shell
docker exec scalardl-samples-scalardl-ledger-1 /async-profiler/bin/asprof -d 60 -e lock -f /tmp/lock.html jps
```

### Wall-clock profiling

```shell
docker exec scalardl-samples-scalardl-ledger-1 /async-profiler/bin/asprof -d 60 -e wall -f /tmp/wall.html jps
```

### JFR format で出力

```shell
docker exec scalardl-samples-scalardl-ledger-1 /async-profiler/bin/asprof -d 60 -f /tmp/profile.jfr jps
```

## Example: profiling during benchmark

1. コンテナを起動する。

```shell
docker-compose -f postgres/docker-compose-ledger.yml up -d
```

2. ベンチマークのデータロードを実行する（クライアント側）。

```shell
./kelpie-1.2.4/bin/kelpie --config ycsb-benchmark-config.toml --only-pre
```

3. プロファイラを開始する（Ledger サーバー側）。

```shell
docker exec scalardl-samples-scalardl-ledger-1 /async-profiler/bin/asprof start -e cpu -f /tmp/profile.html jps
```

4. ベンチマークを実行する（クライアント側）。

```shell
./kelpie-1.2.4/bin/kelpie --config ycsb-benchmark-config.toml --except-pre
```

5. プロファイラを停止し、結果を取得する（Ledger サーバー側）。

```shell
docker exec scalardl-samples-scalardl-ledger-1 /async-profiler/bin/asprof stop -f /tmp/profile.html jps
docker cp scalardl-samples-scalardl-ledger-1:/tmp/profile.html ./profile.html
```

6. `profile.html` をブラウザで開いてフレームグラフを確認する。
