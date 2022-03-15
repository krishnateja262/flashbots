# flashbots ⚡🤖

[![Go Reference](https://pkg.go.dev/badge/github.com/lmittmann/flashbots.svg)](https://pkg.go.dev/github.com/lmittmann/flashbots)
[![Go Report Card](https://goreportcard.com/badge/github.com/lmittmann/flashbots)](https://goreportcard.com/report/github.com/lmittmann/flashbots)
[![Latest Release](https://img.shields.io/github/v/release/lmittmann/flashbots?color=007d9c)](https://github.com/lmittmann/flashbots/releases)

Package flashbots implements RPC API bindings for the Flashbots relay and
[mev-geth](https://github.com/flashbots/mev-geth) for use with the [`w3`](https://github.com/lmittmann/w3)
package.


## Install

```
go get github.com/lmittmann/flashbots
```


## Getting Started

Connect to the Flashbots relay. The [`AuthTransport`](https://pkg.go.dev/github.com/lmittmann/flashbots#AuthTransport)
adds the `X-Flashbots-Signature` header to every request from the client.

```go
// Private key for request authentication
var privKey *ecdsa.PrivateKey

// Connect to Flashbots relay
rpcClient, err := rpc.DialHTTPWithClient(
	"https://relay.flashbots.net",
	&http.Client{
		Transport: flashbots.AuthTransport(privKey),
	},
)

// Create w3 client form rpc client
client := w3.NewClient(rpcClient)
defer client.Close()
```

Send a bundle to the Flashbots relay.

```go
var (
	bundle types.Transactions // list of signed transactions

	bundleHash common.Hash
)

err := client.Call(
	flashbots.SendBundle(&flashbots.SendBundleRequest{
		Transactions: bundle,
		BlockNumber:  big.NewInt(999_999_999),
	}).Returns(&bundleHash),
)
```

Note that the Flashbots relay does not support batch requests. Thus, sending
more than one request in `Client.Call` will result in a server error.


## RPC Methods

List of supported RPC methods.

| Method                     | Go Code
| :------------------------- | :-------
| `eth_sendBundle`           | `flashbots.SendBundle(r *flashbots.SendBundleRequest).Returns(bundleHash *common.Hash)`
| `eth_callBundle`           | `flashbots.CallBundle(r *flashbots.CallBundleRequest).Returns(resp *flashbots.CallBundleResponse)`
| `flashbots_getUserStats`   | `flashbots.UserStats(blockNumber *big.Int).Returns(resp *flashbots.UserStatsResponse)`
| `flashbots_getBundleStats` | `flashbots.BundleStats(bundleHash common.Hash, blockNumber *big.Int).Returns(resp *flashbots.BundleStatsResponse)`
