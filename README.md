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
#### 安装依赖包
    sudo apt install git build-essential ufw curl jq snapd --yes

#### 源码构建
    git clone https://github.com/babylonchain/babylon.git
    cd babylon
    git checkout v0.7.2
    make install

    sudo cp $HOME/go/bin/babylon /usr/local/bin

#### 初始化节点目录
    babylond config keyring-backend test
    babylond config chain-id bbn-test-2
    babylond init $NODENAME --chain-id bbn-test-2

#### 获得创世文件
    wget https://github.com/babylonchain/networks/raw/main/bbn-test-2/genesis.tar.bz2
    tar -xjf genesis.tar.bz2 && rm genesis.tar.bz2
    mv genesis.json ~/.babylond/config/genesis.json

#### 添加种子节点和持久对等节点
    SEEDS="8da45f9ff83b4f8dd45bbcb4f850999637fbfe3b@seed0.testnet.babylonchain.io:26656,4b1f8a774220ba1073a4e9f4881de218b8a49c99@seed1.testnet.babylonchain.io:26656"
    PEERS="88bed747abef320552d84d02947d0dd2b6d9c71c@babylon-testnet.nodejumper.io:44656"
    sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.babylond/config/config.toml
    sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.babylond/config/config.toml
    
    sed -i 's|^pruning *=.*|pruning = "custom"|g' $HOME/.babylond/config/app.toml
    sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $HOME/.babylond/config/app.toml
    sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $HOME/.babylond/config/app.toml
    sed -i 's|^snapshot-interval *=.*|snapshot-interval = 0|g' $HOME/.babylond/config/app.toml
    sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001ubbn"|g' $HOME/.babylond/config/app.toml
    sed -i 's|^network *=.*|network = "mainnet"|g' $HOME/.babylond/config/app.toml
    sed -i 's|^checkpoint-tag *=.*|checkpoint-tag = "bbn0"|g' $HOME/.babylond/config/app.toml

#### 启动与检查节点    
    screen -S baby
    babylond start --x-crisis-skip-assert-invariants

    babylond status
<img width="618" alt="微信截图_20230323165340" src="https://user-images.githubusercontent.com/100336530/227151630-08da75e3-5876-4d0b-b19c-58e2afaa302b.png">

    babylond status 2>&1 | jq .SyncInfo

#### 创建帐号
    babylond --keyring-backend test keys add <your-key-name>

#### 获得测试币
##### 加入Discord
    https://discord.com/invite/babylonchain
    
##### 进入频道接水
    输入如下命令，钱包可使用终端命令生成，现测每个DC间隔5小时可再次领取，每次接水获得100ubbn
    
    !faucet <your-address>
    
<img width="777" alt="微信截图_20230325113634" src="https://user-images.githubusercontent.com/100336530/227689993-077b9389-0004-47a9-832d-a261e3996fef.png">

    查询帐户余额：
    babylond q bank balances $(babylond keys show 钱包名称 -a)

#### 成为验证者
##### 创建 BLS 密钥
    验证者应在每个纪元结束时提交 BLS 签名。 为此，验证者需要有一个 BLS 密钥对来签署信息。创建 BLS 密钥后，需要重新启动节点加载密钥。
    babylond create-bls-key $(babylond keys show 钱包名称 -a)
    
##### 修改配置
    vim ~/.babylond/config/client.toml
    设置正在使用的keyring-backend：keyring-backend = "test"

    vim ~/.babylond/config/app.toml
    指定验证器提交BLS 签名交易的密钥的名称：key-name = "<your-key-name>"

    vim ~/.babylond/config/config.toml
    指定验证器在一个新的高度时提交区块之前需要等待多长时间才能开始：timeout_commit = "10s"

##### 创建验证器
    babylond tx checkpointing create-validator \
    --amount="1000000ubbn" \
    --pubkey=$(babylond tendermint show-validator) \
    --moniker="My Validator" \
    --chain-id=bbn-test-2 \
    --gas="100000" \
    --gas-adjustment=1.2 \
    --gas-prices="0.003ubbn" \
    --keyring-backend=test \
    --from=<your-key-name> \
    --commission-rate="0.10" \
    --commission-max-rate="0.20" \
    --commission-max-change-rate="0.01" \
    --min-self-delegation="1" \
    -y

##### 验证验证节点
    在Babylond只有在纪元结束后才能成为验证者。 对于测试网，一个纪元持续大约 30 分钟。要验证是否已成为验证人，请先找到验证人地址：
    babylond q staking validator $(babylond keys show 钱包名称 --bech val -a)

##### 转账
    babylond tx bank send $from_address $to_address $额度ubbn --gas 80000 --fees 300ubbn -y

##### 增加质押
    babylond tx staking delegate $Validator_Address $额度ubbn --from $WALLET --fess 300ubbn -y























