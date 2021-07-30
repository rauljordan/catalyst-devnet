# Catalyst Devnet Prysm Setup

```
git clone https://github.com/rauljordan/catalyst-devnet && cd catalyst-devnet
```

## Setup Genesis State and Validator Keystores

Install tools for generating eth2 genesis state and keystores 
```
# Install eth2-testnet-genesis tool (Go 1.16+ required)
go install github.com/protolambda/eth2-testnet-genesis@v0.0.1
# Install eth2-val-tools
go install github.com/protolambda/eth2-val-tools@latest
```

Run the genesis state generator

```
eth2-testnet-genesis merge \
  --eth1-config "./eth1_config.json" \
  --eth2-config "./eth2_config.yaml" \
  --mnemonics genesis_validators.yaml \
  --state-output "./data/genesis.ssz" \
  --tranches-dir "./data/tranches"
```

Run the keystores generator from a pre-determined mnemonic

```
eth2-val-tools keystores \
  --out-loc "keystores" \
  --prysm-pass="foobar" \
  --source-min=0 \
  --source-max=64 \
  --source-mnemonic="lumber kind orange gold firm achieve tree robust peasant april very word ordinary before treat way ivory jazz cereal debate juice evil flame sadness"
```

## Build and Configure Catalyst

Clone and build catalyst go-ethereum

```
git clone https://github.com/ethereum/go-ethereum.git && cd go-ethereum
go build -o ../build/bin/catalyst ./cmd/geth
cd ../
```

Initialize catalyst from an eth1 configuration JSON file

```
mkdir -p data/catalyst
./build/bin/catalyst --datadir "./data/catalyst" init "./eth1_config.json"
```

## Build Prysm

Install dependencies for build

```
bazel
```

Clone and build from source
```
git clone -b merge https://github.com/prysmaticlabs/prysm.git && cd prysm
bazel build //beacon-chain:beacon-chain
bazel build //validator:validator
cd ../
```

## Run Catalyst

Run a catalyst node on localhost

```
# block proposal rewards/fees go to the below 'etherbase'.
./build/bin/catalyst \
  --catalyst \
  --rpc \
  --rpcapi net,eth,consensus \
  --nodiscover \
  --miner.etherbase 0x1000000000000000000000000000000000000000 \
  --datadir "./data/catalyst"
```

## Run Prysm Beacon and Validator

Set a few environment variables of file paths

```
export PATH_TO_ETH2_CONFIG=$(pwd)/eth2_config.yaml
export PATH_TO_ETH2_DATA=$(pwd)/data/prysm
export PATH_TO_ETH2_GENESIS=$(pwd)/data/genesis.ssz
export PATH_TO_ETH2_KEYSTORES=$(pwd)/keystores/prysm
```

Run the Prysm beacon

```
bazel run //beacon-chain --define=ssz=minimal -- \
 --datadir=$PATH_TO_ETH2_DATA \
 --min-sync-peers=0 \
 --http-web3provider=http://127.0.0.1:8545 \
 --bootstrap-node= \
 --chain-config-file=$PATH_TO_ETH2_CONFIG \
 --genesis-state=$PATH_TO_ETH2_GENESIS \
 --accept-terms-of-use
 ```

The bootstrap node option is empty to disable any type of networking.

Finally, run the validator client

```
bazel run //validator --define=ssz=minimal -- \
 --wallet-dir=$PATH_TO_ETH2_KEYSTORES \
 --chain-config-file=$PATH_TO_ETH2_CONFIG
```
