# solana wallet on svelte app

Implement solana wallet adapter using svelte

## Background

1. https://github.com/svelte-on-solana/wallet-adapter hasn't been updated for 2 years
2. https://github.com/portalpayments/svelte-on-solana-wallet-adapter hasn't been updated for 1 years

I want to integrate solana wallet on my svelte app, but these 2 projects are outdated, 
So I made this project to learn how solana wallets works on a web frontend project.

## Features

1. Svelte5 support
2. Use the popover api to reduce js to display/hide a modal

## Code Exaplain

`walletStore.ts` contains the core logic about the connected wallet state. all the heavy stuff is done here.

`WalletProvider.svelte` init the walletStore so we can use in other pages

`WalletMultiButton.svelte` If connected a wallet, click this button will see 2 operation menu, if not, click to open a modal with installed wallet list

`Transfer.svelte` A component to do transfer to test the wallet is connected and can sign and send transactions.

## TODO

- [ ] Fix styles for the popovers
