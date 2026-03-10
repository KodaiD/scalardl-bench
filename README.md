# Getting Started

## Setup

```shell
echo "SCALARDL_VERSION=3.12.2" > postgres/.env
```

```shell
echo "YOUR_GHCR_PAT" | docker login ghcr.io -u YOUR_GITHUB_USERNAME --password-stdin
```

On the ledger side:

```shell
docker compose -f postgres/docker-compose-ledger.yml up -d
```

On the auditor side:

```shell
docker compose -f postgres/docker-compose-auditor.yml up -d
```

On the client side:

```shell
sudo apt update && sudo apt install -y openjdk-8-jre openjdk-8-jdk
```

```shell
git clone https://github.com/scalar-labs/scalardl-samples
```

```shell
git clone https://github.com/scalar-labs/scalardl-benchmarks
cd scalardl-benchmarks
```

```shell
cp -r ../scalardl-samples/fixture ./
```

```shell
./gradlew shadowJar
```

```shell
sed -i 's|config_file = "/<PATH_TO>/client.properties"|config_file = "./fixture/client.properties"|' ycsb-benchmark-config.toml
```

```shell
wget https://github.com/scalar-labs/kelpie/releases/download/1.2.4/kelpie-1.2.4.zip
unzip kelpie-1.2.4.zip
```

## Run the benchmark

1. Set the benchmark configuration in `ycsb-benchmark-config.toml`.

```
[ycsb_config]
record_count = 1000000
payload_size = 100
ops_per_tx = 2
workload = "C"
load_concurrency = 16
load_batch_size = 1
```

or

```
[ycsb_config]
record_count = 1000000
payload_size = 100
ops_per_tx = 1
workload = "F"
load_concurrency = 16
load_batch_size = 1
```

```
[common]
concurrency = 4
run_for_sec = 300
ramp_for_sec = 60
```

2. Run the benchmark:

```shell
./kelpie-1.2.4/bin/kelpie --config ycsb-benchmark-config.toml --only-pre
```

```shell
./kelpie-1.2.4/bin/kelpie --config ycsb-benchmark-config.toml --except-pre
```
