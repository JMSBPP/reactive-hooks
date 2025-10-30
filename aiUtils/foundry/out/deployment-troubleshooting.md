# Forge Script Deployment Troubleshooting

## Error Analysis

The deployment script is failing with the following error:

```
Error: server returned an error response: error code -32603: upstream responded emptyish: "0x0", data: {"code":"ErrUpstreamsExhausted","message":"all upstream attempts failed (1 upstream missing data)","details":{"hedges":0,"upstreams":1,"durationMs":11,"projectId":"lasna","networkId":"evm:5318007","method":"eth_getTransactionCount","attempts":3,"retries":2},"cause":[{"code":"ErrEndpointMissingData","message":"remote endpoint does not have this data","details":{"finalizedBlock":677842,"maxAvailableRecentBlocks":0,"upstreamId":"prq-dc1-0","latestBlock":677980},"cause":"upstream responded emptyish: \"0x0\""}]}
```

## Root Causes

### 1. RPC Endpoint Issues
- **Problem**: The `reactive-testnet` RPC endpoint is experiencing connectivity issues
- **Evidence**: "upstream responded emptyish: '0x0'" and "all upstream attempts failed"
- **Impact**: Cannot fetch transaction count or interact with the network

### 2. Script Dependencies
- **Problem**: The `DeployReactivePluginFactory.s.sol` script references `DeployDoubleSwapReactiveHook.run()` but doesn't import it
- **Evidence**: Line 30 calls `DeployDoubleSwapReactiveHook.run()` without proper import
- **Impact**: Compilation error when trying to deploy

### 3. Missing Environment Variables
- **Problem**: Script requires `CALLBACK_SENDER` environment variable for `DeployDoubleSwapReactiveHook`
- **Evidence**: Line 26 in `DeployDoubleSwapReactiveHook.s.sol` calls `vm.envAddress("CALLBACK_SENDER")`
- **Impact**: Runtime error if environment variable is not set

## Solutions

### Solution 1: Fix RPC Endpoint Issues

#### Option A: Use Alternative RPC Endpoint
```bash
# Try using a different RPC endpoint or check if the current one is working
curl -X POST -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1}' \
  https://lasna-rpc.rnk.dev/
```

#### Option B: Add RPC Endpoint Fallback
Update `foundry.toml`:
```toml
[rpc_endpoints]
reactive-testnet = "https://lasna-rpc.rnk.dev/"
reactive-testnet-backup = "https://alternative-rpc-endpoint.com/"
```

### Solution 2: Fix Script Dependencies

Update `script/DeployReactivePluginFactory.s.sol`:

```solidity
// Add this import at the top
import {DeployDoubleSwapReactiveHook} from "./DeployDoubleSwapReactiveHook.s.sol";

// Fix the function call on line 30
if (reactiveHook == address(0x00)) {
    (IReactiveHooks reactiveHookContract, uint256 deploymentBlock) = DeployDoubleSwapReactiveHook.run();
    reactiveHook = address(reactiveHookContract);
}
```

### Solution 3: Set Required Environment Variables

```bash
export PRIVATE_KEY="your_private_key_here"
export CALLBACK_SENDER="0xYourCallbackSenderAddress"
```

### Solution 4: Test Network Connectivity

```bash
# Test if the RPC endpoint is responding
forge script script/DeployReactivePluginFactory.s.sol:DeployReactivePluginFactory --dry-run --rpc-url reactive-testnet --chain-id 5318007 -vvvv
```

## Recommended Fix Sequence

1. **First**: Fix the script import issue
2. **Second**: Set environment variables
3. **Third**: Test with dry-run
4. **Fourth**: Try deployment with verbose output

## Alternative Deployment Approach

If the RPC endpoint continues to have issues, consider:

1. **Local Testing**: Deploy to a local Anvil node first
2. **Different Network**: Use a more stable testnet
3. **Manual Deployment**: Deploy contracts individually using Remix or other tools

## Files Modified

- `script/DeployReactivePluginFactory.s.sol` - Add missing import
- `foundry.toml` - Add backup RPC endpoint (optional)
- Environment variables - Set `CALLBACK_SENDER`

## Verification Commands

```bash
# Check if RPC is working
forge script --dry-run --rpc-url reactive-testnet --chain-id 5318007

# Deploy with fixed script
forge script script/DeployReactivePluginFactory.s.sol:DeployReactivePluginFactory --legacy --broadcast --rpc-url reactive-testnet --chain-id 5318007 -vvvv
```
