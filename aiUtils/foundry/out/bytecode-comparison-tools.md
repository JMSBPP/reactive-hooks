# Foundry Bytecode Comparison and Outdated Contract Detection Tools

## Overview

Foundry provides several tools and approaches to detect if deployed contracts have outdated bytecode compared to your local source code. This is crucial for ensuring that deployed contracts match your current codebase and haven't been compromised or accidentally deployed with outdated versions.

## Built-in Foundry Tools

### 1. `forge verify-bytecode` - Direct Bytecode Verification

**Purpose**: Verifies deployed bytecode against source code on Etherscan/block explorers.

**Usage**:
```bash
forge verify-bytecode <ADDRESS> <CONTRACT> --rpc-url <RPC_URL> --etherscan-api-key <API_KEY>
```

**Key Features**:
- Compares deployed bytecode with locally compiled bytecode
- Supports multiple verification providers (Etherscan, Sourcify, Blockscout, OKLink)
- Can verify at specific block heights
- Handles constructor arguments automatically

**Example**:
```bash
forge verify-bytecode 0x1234...5678 src/ReactivePluginFactory.sol:ReactivePluginFactory \
  --rpc-url reactive-testnet \
  --etherscan-api-key $ETHERSCAN_API_KEY \
  --chain 5318007
```

### 2. `forge verify-contract` - Source Code Verification

**Purpose**: Verifies that deployed contract source code matches your local source.

**Usage**:
```bash
forge verify-contract <ADDRESS> <CONTRACT> --rpc-url <RPC_URL> --etherscan-api-key <API_KEY>
```

**Key Features**:
- Submits source code to block explorer for verification
- Automatically handles constructor arguments
- Supports flattening for complex contracts
- Can guess constructor arguments from on-chain data

**Example**:
```bash
forge verify-contract 0x1234...5678 src/ReactivePluginFactory.sol:ReactivePluginFactory \
  --rpc-url reactive-testnet \
  --etherscan-api-key $ETHERSCAN_API_KEY \
  --chain 5318007 \
  --constructor-args $(cast abi-encode "constructor(address,uint256)" $REACTIVE_HOOK $CHAIN_ID)
```

### 3. `forge verify-check` - Verification Status Check

**Purpose**: Checks the status of previously submitted verification requests.

**Usage**:
```bash
forge verify-check <VERIFICATION_ID> --etherscan-api-key <API_KEY>
```

## Third-Party Tools

### 1. Diffyscan - GitHub vs Etherscan Comparison

**Repository**: [lidofinance/diffyscan](https://github.com/lidofinance/diffyscan)

**Purpose**: Compares your GitHub repository against deployed contracts on Etherscan to detect discrepancies.

**Features**:
- Compares bytecode between GitHub and Etherscan
- Detects outdated deployments
- Supports multiple networks
- Command-line interface for automation

**Installation**:
```bash
pip install diffyscan
```

**Usage**:
```bash
diffyscan --repo owner/repo --address 0x1234...5678 --network mainnet
```

### 2. Custom Scripts Using Foundry Cheatcodes

**Using `getDeployedCode` Cheatcode**:

```solidity
// In a test file
contract BytecodeComparisonTest is Test {
    function testBytecodeMatch() public {
        // Get deployed bytecode
        bytes memory deployedBytecode = getDeployedCode("src/ReactivePluginFactory.sol:ReactivePluginFactory");
        
        // Get local bytecode (from compilation)
        bytes memory localBytecode = getCode("src/ReactivePluginFactory.sol:ReactivePluginFactory");
        
        // Compare bytecodes
        assertEq(deployedBytecode, localBytecode, "Deployed bytecode does not match local bytecode");
    }
}
```

**Required foundry.toml configuration**:
```toml
fs_permissions = [{ access = "read", path = "./out" }]
```

## Practical Implementation Strategies

### 1. Automated Bytecode Verification Script

Create a script to automatically check all deployed contracts:

```solidity
// script/VerifyBytecodes.s.sol
contract VerifyBytecodes is Script {
    function run() public {
        // List of deployed contracts to verify
        address[] memory contracts = new address[](2);
        contracts[0] = 0x1234...5678; // ReactivePluginFactory
        contracts[1] = 0xabcd...efgh; // MockDoubleSwapReactiveHook
        
        string[] memory contractPaths = new string[](2);
        contractPaths[0] = "src/ReactivePluginFactory.sol:ReactivePluginFactory";
        contractPaths[1] = "test/mocks/MockDoubleSwapReactiveHook.sol:MockDoubleSwapReactiveHook";
        
        for (uint i = 0; i < contracts.length; i++) {
            verifyContract(contracts[i], contractPaths[i]);
        }
    }
    
    function verifyContract(address contractAddr, string memory contractPath) internal {
        // Implementation to verify each contract
    }
}
```

### 2. CI/CD Integration

Add bytecode verification to your deployment pipeline:

```yaml
# .github/workflows/verify-deployments.yml
name: Verify Deployments
on:
  push:
    branches: [main]

jobs:
  verify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install Foundry
        uses: foundry-rs/foundry-toolchain@v1
      - name: Verify Bytecode
        run: |
          forge verify-bytecode $CONTRACT_ADDRESS $CONTRACT_PATH \
            --rpc-url $RPC_URL \
            --etherscan-api-key $ETHERSCAN_API_KEY
```

### 3. Monitoring Script for Outdated Contracts

```bash
#!/bin/bash
# scripts/check-outdated-contracts.sh

CONTRACTS=(
    "0x1234...5678:src/ReactivePluginFactory.sol:ReactivePluginFactory"
    "0xabcd...efgh:test/mocks/MockDoubleSwapReactiveHook.sol:MockDoubleSwapReactiveHook"
)

for contract in "${CONTRACTS[@]}"; do
    IFS=':' read -r address path <<< "$contract"
    echo "Checking $address..."
    
    if ! forge verify-bytecode $address $path --rpc-url $RPC_URL --etherscan-api-key $API_KEY; then
        echo "❌ Contract $address has outdated bytecode!"
        # Send alert or create issue
    else
        echo "✅ Contract $address is up to date"
    fi
done
```

## Best Practices

### 1. Regular Verification
- Run bytecode verification after every deployment
- Include verification in CI/CD pipelines
- Set up monitoring for critical contracts

### 2. Version Tracking
- Tag your deployments with version numbers
- Store deployment artifacts with version information
- Maintain a registry of deployed contract versions

### 3. Automated Alerts
- Set up alerts for bytecode mismatches
- Monitor verification failures
- Track deployment consistency

### 4. Documentation
- Document all deployed contract addresses
- Maintain a changelog of contract updates
- Keep verification commands in deployment scripts

## Troubleshooting Common Issues

### 1. Constructor Arguments Mismatch
```bash
# Use --guess-constructor-args to automatically detect
forge verify-contract $ADDRESS $CONTRACT --guess-constructor-args

# Or provide explicit constructor arguments
forge verify-contract $ADDRESS $CONTRACT --constructor-args $(cast abi-encode "constructor(uint256)" 123)
```

### 2. Compiler Version Mismatch
```bash
# Specify exact compiler version
forge verify-contract $ADDRESS $CONTRACT --compiler-version 0.8.26
```

### 3. Library Linking Issues
```bash
# Provide library addresses
forge verify-contract $ADDRESS $CONTRACT --libraries "lib1:0x1234...5678,lib2:0xabcd...efgh"
```

## Integration with Your Project

For your Reactive Hook Plugins project, you can:

1. **Add verification to deployment scripts**:
   ```bash
   # After deployment
   forge verify-contract $REACTIVE_PLUGIN_FACTORY_ADDRESS src/ReactivePluginFactory.sol:ReactivePluginFactory \
     --rpc-url reactive-testnet \
     --etherscan-api-key $ETHERSCAN_API_KEY \
     --chain 5318007
   ```

2. **Create a verification script**:
   ```bash
   # scripts/verify-deployments.s.sol
   forge script scripts/verify-deployments.s.sol:VerifyDeployments --rpc-url reactive-testnet
   ```

3. **Set up monitoring**:
   - Use the monitoring script to check contracts regularly
   - Integrate with your existing CI/CD pipeline
   - Set up alerts for bytecode mismatches

## Conclusion

Foundry provides robust tools for detecting outdated bytecode in deployed contracts. By combining built-in verification commands with third-party tools like Diffyscan and custom monitoring scripts, you can ensure that your deployed contracts always match your current codebase. This is especially important for critical infrastructure contracts like your Reactive Plugin Factory.

The key is to integrate these tools into your development workflow and make bytecode verification a standard part of your deployment process.
