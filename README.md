# multichain-relayer-server
Pagoda Implementation of Multichain Relayer Server facilitating cross chain transactions enabling Chain Abstraction.

## Technical System Design
Below is a Design Diagram of the entire multichain relayer system.

- The [gas station repo](https://github.com/near/multichain-gas-station-contract) and the [mpc signing service](https://github.com/near/mpc-recovery/tree/main/contract) are in the green box which happens on NEAR.
- This multichain relayer server repo focuses on the purple/blue Multichain Relayer Core Backend Services Box in the middle and the connections to the XChain systems in the red box via RPCs.
- The XChain Settlement that's happening in the yellow box is currently manual and will be automated in the future.

![multichain_relayer_technical_design.png](multichain_relayer_technical_design.png)

## Technical System Description
1. The wallet sends a NEAR transaction to the gas station contract that contains 2 actions
   1. A transfer of NEAR (or FT Transfer in the future) to cover gas on the foreign chain
   2. A `create_transaction` function call to the gas station contract `canhazgas.testnet` containing the unsigned foreign chain transaction to be signed by the MPC signing service, assuming the unsigned transaction passes validation.
2. The Gas Station Contract calls the MPC signing service to sign both a funding transaction, which ensures the user's foreign chain account has sufficient gas to execute the desired transaction, and signs the unsigned foreign chain transaction.
3. Upon receipt of both the signed transactions, the Gas Station Contract emits an event which is picked up by the near lake indexer, which then calls the `/send_funding_and_user_signed_txns` with the 2 signed transactions from the indexer.
4. The multichain relayer server sends the funding transaction to the foreign chain RPC to fund the user's account with gas.
5. After the gas funding transaction is confirmed, the multichain relayer server calls the foreign chain RPC again to send the signed transaction initiated by the user in step 1.
6. The Cross Chain Settlement takes care of selling the extra NEAR being sent to the gas station contract for gas tokens on foreign chains as well as bridging the tokens to the other chains. This process is currently manual, but will be automated in partnership with market makers in the future.

## Supported Chains
- BSC testnet
- BSC Mainnet (March 31 2024)
- Solana Testnet (March 31 2024)
- Solana Mainnet (March 31 2024)
- More chains coming soon!

## Multichain Relayer Description
- The main function of this server is interfacing with foreign chain RPCs sending both presigned funding transactions to cover gas and the actual presigned transaction once the funding is done. 
- The Multichain relayer is meant to be deployed alongside the [gas-station-event-indexer](https://github.com/near/gas-station-event-indexer) on the same server so that the gas station event indexer can call the multichain relayer server via IPC instead of having to send the request over the network introducing extra latency to the system.
- Although the multichain relayer is a server in current design of this system, the goal is to package this as a library that can be called on the client side of the wallet. This will make the system more decentralized.
 

## Functionality
1. `/send_funding_and_user_signed_txns` which handles both
   1. Funding the user's xchain account with gas from the paymaster treasury account, which is provided as a raw signed transaction
   2. Sending the user's raw signed transaction (in hexadecimal in EVM case) after the funding transaction has been confirmed on the foreign chain
2. (MVP only) `/get_balance_for_account` endpoint that gets the native gas token (BNB) balance for an account. AKA Poor man's indexing

## Expected Time


## Future Directions
### Scale:
- Moar Chainzzz
- Better Capital Efficiency with ~real-time settlement using Orderly + professional MMs
### Decentralization:
- Replace Multichain Relayer Server with Client Side Libraries
- Txn Trie Inclusion Proofs
### Broader Wallet Support:
- Provide Chain Abstraction Services to wallets that don’t implement MPC
### Intent Relayers
- AI is able to execute appropriate transactions on behalf of the user getting them the best price across all chains. 
  - This involves aggregating and indexing FT, NFT market dapps.
  - i.e. User has **_SWEAT_** they'd like to **_swap_** for **_BTC_**. From these keywords, AI calls multichain relayer services to execute this action on their behalf. The actual execution may look like the user's SWEAT is sold on Orderly on the NEAR chain and they end up receiving WBTC on SOL account. 


