# Testnet Proposals

Let's say that you want to submit a proposal on `testnet`. Because there is a short period of voting time for the proposals, we recommend lowering the deposit on the proposal to not make the proposal go into voting stage directly. Basically, it should be slightly less than `min_deposit` value.

Once you submit the proposal, you should reach out to the team:

1. Join the Titan Discord server and find the relevant channel.
2. Join the Titan Developer Telegram channel.

Here is an example for the `GrantProviderPrivilegeProposal`

```bash
 titand oracle grant-provider-privilege-proposal YOUR_PROVIDER \
  YOUR_ADDRESS_HERE \
  --title="TITLE OF THE PROPOSAL" \
  --description="Registering PROVIDER as an oracle provider" \
  --chain-id=titan_18889-1 \
  --from=local_key \
  --node=https://titan-testnet-rpc.titanlab.io:443 \
  --gas-prices=160000000atkx \
  --gas=20000000 \
  --deposit="40000000000000000000atkx" <-- use this amount
```

