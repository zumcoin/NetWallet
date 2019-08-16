# ZUM NetWallet

![screenshot](/docs/screenshot-netwallet-login.png)

## Setup on Ubuntu 16.04+
Install the required packages.  
`sudo apt install git postgresql postgresql-contrib redis-server`  
[Install golang-1.10](https://gist.github.com/ndaidong/4c0e9fbae8d3729510b1c04eb42d2a80)

Don't forget to make your GOPATH export persistent.

Install the necessary go libraries
```
go get \
	github.com/gomodule/redigo/redis \
	github.com/julienschmidt/httprouter \
	github.com/lib/pq \
	github.com/opencoff/go-srp \
	github.com/ulule/limiter \
	github.com/ulule/limiter/drivers/middleware/stdlib \
	github.com/ulule/limiter/drivers/store/memory \
	github.com/dchest/captcha
```

Clone the ZUM NetWallet repo in your ${GOPATH}/src.

#### Postgres Setup
[Configure postgres user](https://www.linode.com/docs/databases/postgresql/how-to-install-postgresql-on-ubuntu-16-04/)  

Setup user database  
`~$ cat user_db.sql | psql -U <username> -h <host>`  
Setup transactions database  
`~$ cat transaction_db.sql | psql -U <username> -h <host>`

#### Setup ZumCoin service
Run this once to generate a wallet container.
`~$ ./zum-service --container-file <container name> -p <password> -g`  

Point zum-service at an existing daemon like this
`~$ ./zum-service --rpc-password <rpc password> --container-file <container name> -p <container password> -d --daemon-address <daemon DNS or IP address> --daemon-port <daemon port>`

#### Start redis-server

#### Configure and start run scripts
Edit these files:
* services/main/run.sh  
```bash
#!/usr/bin/env bash
HOST_URI='https://netwallet.zumcoin.org' \ # Web wallet address
HOST_PORT=':8080' \ # Internal server port
USER_URI='http://localhost:8081' \ # Internal requests to user api
WALLET_URI='http://localhost:8082' \ # Internal requests to wallet api
go run main.go utils.go
```
* services/wallet/run.sh  
```bash
#!/usr/bin/env bash
DB_USER=<postgres username> \ # Postgres DB username, NOT system account username
DB_PWD=<postgres password> \ # Postgres DB password, NOT system account password
HOST_URI='http://localhost' \ # Internal wallet api
HOST_PORT=':8082' \ # Internal wallet api port
RPC_PWD=<zum-service RPC password>  \ # Your zum-service RPC password
RPC_PORT=':17070' \ # Your zum-service RPC port
go run wallet.go utils.go
```
* services/user/run.sh  
```bash
#!/usr/bin/env bash
DB_USER=<postgres username> \ # Postgres DB username, NOT system account username
DB_PWD=<postgres password> \ # Postgres DB password, NOT system account password
HOST_URI='http://localhost' \ # Internal user api
HOST_PORT=':8081' \ # Internal user api port
WALLET_URI='http://localhost:8082' \ # Internal wallet api
go run users.go utils.go
```

`~$ cd services/main ; ./run.sh & disown`  
`~$ cd services/wallet ; ./run.sh & disown`  
`~$ cd services/user ; ./run.sh & disown`  



## Dependencies
* Redis
* Postgresql
* Go
* ZumCoin wallet daemon
