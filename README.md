# Mariadb Galera Cluster

### 1. Create .env from tempage and edit .env
```bash
cp .env.default .env
```
```bash
nano .env
```

### 2. Generate self signed certificate for mariadb
```bash
mkdir -p /apps/certs/mariadb && cd /apps/certs/mariadb
```

`--- CA Certificate ---`
```bash
openssl genrsa 2048 > ca-key.pem
```
```bash
openssl req -new -x509 -nodes -days 36500 \
-key ca-key.pem -out ca-cert.pem
```
`--- Server Certificate ---`
```bash
openssl req -newkey rsa:2048 -days 36500 \
-nodes -keyout server-key.pem -out server-req.pem
```
```bash
openssl rsa -in server-key.pem -out server-key.pem
```
```bash
openssl x509 -req -in server-req.pem -days 36500 \
-CA ca-cert.pem -CAkey ca-key.pem -set_serial 01 \
-out server-cert.pem
```
`--- Client Certificate ---`
```bash
openssl req -newkey rsa:2048 -days 36500 \
-nodes -keyout client-key.pem -out client-req.pem
```
```bash
openssl rsa -in client-key.pem -out client-key.pem
```
```bash
openssl x509 -req -in client-req.pem -days 36500 \
-CA ca-cert.pem -CAkey ca-key.pem -set_serial 01 \
-out client-cert.pem
```
`--- Verifying the Certificates ---`
```
openssl verify -CAfile ca-cert.pem \
server-cert.pem client-cert.pem
```

### 3. Change owner certificate
```bash
chown -R 1001:1001 *
```

### 4. Copy mariadb certificate to node 2, 3
```bash
ssh <user>@<ip-node2> "mkdir -p /apps/certs/mariadb/"
scp -pr /apps/certs/mariadb/* <user>@<ip-node2>:/apps/certs/mariadb/
ssh <user>@<ip-node2> "chown -R 1001:1001 /apps/certs/mariadb/"
```
```bash
ssh <user>@<ip-node3> "mkdir -p /apps/certs/mariadb/"
scp -pr /apps/certs/mariadb/* <user>@<ip-node3>:/apps/certs/mariadb/
ssh <user>@<ip-node3> "chown -R 1001:1001 /apps/certs/mariadb/"
```

### 5. Create dir
`node 1`
```
mkdir data_node1 && chown 1001:1001 data_node1
```
`node 2`
```
mkdir data_node2 && chown 1001:1001 data_node2
```
`node 3`
```
mkdir data_node3 && chown 1001:1001 data_node3
```

### 6. Run Docker
`node 1`
```bash
docker-compose -f mariadb-galera-1.yml up -d
```
`node 2`
```bash
docker-compose -f mariadb-galera-2.yml up -d
```
`node 3`
```bash
docker-compose -f mariadb-galera-3.yml up -d
```
