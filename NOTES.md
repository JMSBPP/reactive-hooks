



## Claims
- diamond provides modulability on systems and reactive does the reactivity of their events

```solidity
event DiamondCut(FacetCut[] _diamondCut, address _init, bytes _calldata);
    // Records all function changes to a diamond
```
- This is like derivatives and reactive is

$$
\frac{d\,\text{system}_x}{d\,\text{sub-system}_{xy}} = a \implies \frac{d\,\text{system}_y}{da}
$$



## Hooks Specific 
- hooks are __facets__ of pools
- Should hooks be stateless, statefull ?
- PoolActions have immutable actions
- PoolActions can be subdivided subejetc to the client
 - Liquidity providers -> PoolLiquidity --> PoolLiquidityState
					--> Immutable actions/ invariants
					--> Mutable services/facets

---> BeaCON

If you own the liquditiy managerment of a pool you want to have a mechanism to upgrade such functinaluity


> It allows you to deploy a new implementation(Hook) contract and upgrade all proxies (Pools attached to it) simultaneously.

## Resources
	- https://github.com/arka-kxqi/reactfi: ReactDEFI already built (UI can be used on our's)
	- design idea on card
        - https://reacdefi.app/about: website
	- https://blog.reactive.network/reacdefi-for-on-chain-stop-orders-and-beyond/: blog

    - https://docs.solarity.dev/docs/getting-started/Overview: Has a diamond implementation

