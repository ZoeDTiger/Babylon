# Babylon


### 安装 Golang
    cd $HOME
    wget "https://go.dev/dl/go1.19.4.linux-amd64.tar.gz"
    sudo rm -rf /usr/local/go
    sudo tar -C /usr/local -zxvf go1.19.4.linux-amd64.tar.gz

    echo "export GOROOT=/usr/local/go" |  sudo tee -a /etc/profile
    echo "export GOPATH=$HOME/go" |  sudo tee -a /etc/profile
    echo "export PATH=$PATH:/usr/local/go/bin:$GOPATH/bin" |  sudo tee -a /etc/profile
    echo "export GO111MODULE=on" |  sudo tee -a /etc/profile
    echo "export GOPROXY=https://goproxy.cn" |  sudo tee -a /etc/profile

    source /etc/profile
    go version

### 构建并安装Babylon
#### 安装工具包
    sudo apt install git build-essential ufw curl jq snapd --yes

#### 源码构建
    git clone https://github.com/babylonchain/babylon.git
    cd babylon
    git checkout v0.5.0
    make install

    sudo cp $HOME/go/bin/babylon /usr/local/bin.

#### 初始化节点目录
    babylond init $NODENAME --chain-id bbn-test1

#### 获得创世文件
    wget https://github.com/babylonchain/networks/raw/main/bbn-test1/genesis.tar.bz2
    tar -xjf genesis.tar.bz2 && rm genesis.tar.bz2
    mv genesis.json ~/.babylond/config/genesis.json

#### 添加种子节点和持久对等节点
    cd $HOME~/.babylond/config
    vim config.toml

    设置seeds：
    03ce5e1b5be3c9a81517d415f65378943996c864@18.207.168.204:26656,a5fabac19c732bf7d814cf22e7ffc23113dc9606@34.238.169.221:26656,ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@testnet-seeds.polkachu.com:20656













