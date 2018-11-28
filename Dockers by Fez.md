# Guide Built for Ubuntu 18.04 - Docker container

Thanks to Jagerman @Jagerman (Telegram handle) for the alpha 3 code fork and optimizations included, including the wizard to install his packages and the watch-only-wallets download for testnet. https://github.com/jagerman/GraftNetwork  
Thanks to MustDie95 for the initial Docker build on Alpha3 code which I used as a backbone to get this running and tweaked his supervisor config to make the graftnoded and graft_server start automatically once the docker container starts. https://github.com/MustDie95/graft  
Thanks to Tiago S @el_duderino_007 (Telegram handle) for the blockchain download and initail guide for installing Alpha3 code on server.
 
## Resources:
* Github: https://github.com/Fez29/fez-graft-docker.git
* Docker Hub: https://hub.docker.com/r/fez29/graftnoded-jagerman/
 
Note: if you are running as root please exclude any sudo references below:

````bash
sudo apt-get remove docker docker-engine docker.io
sudo apt-get update
sudo apt-get upgrade    # (for all promts just select keep as is) Optional
````

Install Docker dependencies (Copy entire command - all 6 lines and paste)

````bash
sudo apt-get install \
apt-transport-https \
ca-certificates \
curl \
gnupg2 \
software-properties-common
````
 
 Add Docker’s official GPG key:

````bash
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
````

Test success with:

````bash
sudo apt-key fingerprint 0EBFCD88
````

Should return:
````bash
	pub   4096R/0EBFCD88 2017-02-22
	      Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
	uid                  Docker Release (CE deb) <docker@docker.com>
	sub   4096R/F273FCD8 2017-02-22
````

## Add docker repo

Stable repo:
````bash
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
````
Edge repo:
````bash
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic edge"
````

Update apt again and install docker Community Edition:
````bash
sudo apt-get update
sudo apt-get install docker-ce
````

Prepare Docker volume mount:
````bash
sudo mkdir $HOME/.graft
````

Download image and run container after with mounted volume (Check Optional at botom of this guide before continuing)
:
````bash
sudo docker run --name graft -d -v $HOME/.graft:/root/.graft -p 28690:28690 -p 28680:28680 fez29/graftnoded-jagerman:Jagerman-Experiment_fez29
````

Use docker exec to login to the container as root:
````bash
docker exec -ti graft /bin/bash
````

Note that graftnoded and graft_server are automatically started and restarted if they die by by supervisor - See optional if you want to disable/adjust behaviour. 
(Check section in optional regarding downloading watch-only-wallets now if want to action)
Check sync status:
````bash
graftnoded --testnet status
````

Check wallet address for Supernode:
````bash
graft-wallet-cli --wallet-file ~/.graft/supernode/data/stake-wallet/stake-wallet --password "" --testnet --trusted-daemon
````
Record seed and store safely especially on Mainnet (KEEP OFFLINE or written down and never reveal to anyone!)
when in wallet type: seed and follow prompt (no password just press enter)

Once graftnoded fully synced and stake in wallet, kill graft_server process to speed up process.
 Kill graft_server process:
````bash
ps ax | graft_server
````
grab process ID = Furthest to the left of the line showing below:
````bash
	4002 ?        Sl     1:32 graft_server --log-file supernode.log --log-level 1 > out.log 2>&1
````
In this example use = 4002
````bash
kill -9 4002
````
graft_server will start automatically again.

Viewing logs: (-n {150} - 150 equals lines to view – adjust accordingly)

Graft_server logs:
````bash
cd /home/graft-sn/supernode
tail -f -n 150 supernode.log
````
Graftnoded logs from anywhere:
````bash
tail -f -n 150 /$HOME/.graft/testnet/graft.log
````
Checking locally inside container if supernode list is being generated:
Install curl:
````bash
apt update
apt install curl -y
curl 127.0.0.1:28690/debug/supernode_list/0
````

Do not forget to open ports for 28690 and 28680 on VPS or port forward if hosting locally.

Test list from external:
````bash
<external-IP>:28690/debug/supernode_list/0
```` 
## Docker commands

Stop and Start container (use name from docker run command)
````bash
sudo stop graft
sudo start graft
````

To setup docker so that container starts automatically if machine restarted:

On running container after exiting container - just type exit when logged in to exit
````bash
docker container update --restart unless-stopped graft
````
Above command in bold is name used in docker run command


## Optional:
(Beware of using below on mainnet is much safer to download Blockchain youself | also please consult @Jagerman before using 
````bash
any watch-only-wallets downloads on mainnet):
````

Before running the image download and docker run step, download provided blockchain directory.
````bash
apt update
sudo apt install unzip -y
sudo mkdir $HOME/.graft (if not done already)
wget "https://www.dropbox.com/s/b55s59bluvp8s1z/graft_bc_testnet_bkp_17Nov18.zip" -P /tmp
unzip /tmp/graft_bc_testnet_bkp_17Nov18.zip '.graft/testnet/*' -d $HOME/
````

Go back to: Download image and run container after with mounted volume 

## ADVANCED

Recommended Docker container updates
Restart container automatically on reboot of server:
````bash
sudo docker container update --restart unless-stopped graft
````
Limit cpu usage to 2 cores: (adjustable)
````bash
sudo docker container update --cpus 2 graft
````

Download Watch only wallets:
Login to already running container:
````bash
docker exec -ti graft /bin/bash
mkdir -p ~/.graft/supernode/data/{watch-only-wallets,stake-wallet} && cd ~/.graft/supernode/data && curl -s https://rta.graft.observer/lmdb/watch-only-wallets.tar | tar xvf -
````
Kill graft_server process:
````bash
ps ax | grep graft_server
````
grab process ID = Furthest to the left of the line showing below:
````bash
	4002 ?        Sl     1:32 graft_server --log-file supernode.log --log-level 1 > out.log 2>&1
````
In this example use = 4002
````bash
kill -9 4002
````
graft_server will start automatically again.
````bash
cd back to location for supernode logs 
cd/home/graft-sn/supernode
tail -f -n 150 supernode.log to view logs
```` 

To adjust supervisor behaviour:
Install nano:
````bash
apt update
apt install nano -y
cd /etc/supervisor/conf.d/
nano graftnoded.conf
nano graft_server.conf
````
restart container by exiting container:
Type: exit  until you see you have exited the container.
````bash
sudo stop graft
sudo start graft
````

## To build image from scratch:
Install Git:
````bash
apt update
apt install git -y
````

Clone repo:
````bash
git clone https://github.com/Fez29/fez-graft-docker.git
cd fez-graft-docker
````

Build image yourself:
````bash
docker build - < Dockerfile -t <chosen_name>:<chosen_tag>
````
eg mine was built with:
````bash
docker build - < Dockerfile -t fez29/graftnoded-jagerman:Jagerman-Experiment_fez29
````
Then got to Prepare Docker volume mount: section of guide. Or optional section for Blockchain download

