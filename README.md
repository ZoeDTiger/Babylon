# Babylon验证节点部署说明


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
    设置seeds值：
    03ce5e1b5be3c9a81517d415f65378943996c864@18.207.168.204:26656,a5fabac19c732bf7d814cf22e7ffc23113dc9606@34.238.169.221:26656,ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@testnet-seeds.polkachu.com:20656

    vim app.toml
    设置btc-tag：bbn0

#### 启动与检查节点
    screen -S baby
    babylond start

    babylond status
<img width="618" alt="微信截图_20230323165340" src="https://user-images.githubusercontent.com/100336530/227151630-08da75e3-5876-4d0b-b19c-58e2afaa302b.png">

#### 创建帐号
    babylond --keyring-backend test keys add <your-key-name>

#### 获得测试币
##### 加入Discord
    https://discord.com/invite/babylonchain
    
##### 进入频道接水
    输入如下命令，钱包可使用终端命令生成，每天每次接水获得100ubbn
    
    !faucet <your-address>
    
<img width="777" alt="微信截图_20230325113634" src="https://user-images.githubusercontent.com/100336530/227689993-077b9389-0004-47a9-832d-a261e3996fef.png">


#### 成为验证者
##### 创建 BLS 密钥
    验证者应在每个纪元结束时提交 BLS 签名。 为此，验证者需要有一个 BLS 密钥对来签署信息。创建 BLS 密钥后，需要重新启动节点加载密钥。
    babylond create-bls-key <your-address>
    
##### 修改配置
    vim ~/.babylond/config/client.toml
    设置正在使用的keyring-backend：keyring-backend = "test"

    vim ~/.babylond/config/app.toml
    指定验证器提交BLS 签名交易的密钥的名称：key-name = "<your-key-name>"

    vim ~/.babylond/config/config.toml
    指定验证器在一个新的高度时提交区块之前需要等待多长时间才能开始：timeout_commit = "10s"

##### 创建验证器
    babylond tx checkpointing create-validator \
    --amount="10000000ubbn" \
    --pubkey=$(babylond tendermint show-validator) \
    --moniker="My Validator" \
    --chain-id=bbn-test1 \
    --gas="auto" \
    --gas-adjustment=1.2 \
    --gas-prices="0.0025ubbn" \
    --keyring-backend=test \
    --from=<your-key-name> \
    --commission-rate="0.10" \
    --commission-max-rate="0.20" \
    --commission-max-change-rate="0.01" \
    --min-self-delegation="1"

##### 验证验证节点
    在Babylond只有在纪元结束后才能成为验证者。 对于测试网，一个纪元持续大约 30 分钟。要验证是否已成为验证人，请先找到验证人地址：
    
    babylond keys show <your-key-name> -a --bech val



























