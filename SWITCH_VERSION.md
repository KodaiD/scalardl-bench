## Switch Version

```shell
docker-compose -f postgres/docker-compose-ledger.yml down -v
```

or

```shell
docker-compose -f postgres/docker-compose-auditor.yml down -v
```

---

```shell
sudo rm -rf /mnt/disks/ssds/postgres-data
```

```shell
docker system prune -a --volumes
```

---

```shell
cd scalardl
git checkout <TARGET_VERSION>
./gradlew :ledger:docker -x test
cd ../
```

or 

```shell
cd scalardl-enterprise
git checkout <TARGET_VERSION>
./gradlew :auditor:docker -x test
cd ../
```

---

```shell
echo "SCALARDL_VERSION=<VERSION>" > postgres/.env
```

---

```shell
vim postgres/.env
docker-compose -f postgres/docker-compose-ledger.yml up -d
```

or

```shell
vim postgres/.env
docker-compose -f postgres/docker-compose-auditor.yml up -d
``` 
