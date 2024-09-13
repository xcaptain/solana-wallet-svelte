<script lang="ts">
    import { type Adapter } from "@solana/wallet-adapter-base";
    import { walletStore, type Wallet } from "./walletStore";

    const byInstalledStatus = (a: Wallet, b: Wallet) => {
        if (a.readyState === "Installed" && b.readyState !== "Installed") {
            return -1;
        }
        if (a.readyState !== "Installed" && b.readyState === "Installed") {
            return 1;
        }
        return 0;
    };
    $: installedWalletAdaptersWithReadyState = $walletStore.wallets
        .filter((walletAdapterAndReadyState) => {
            return walletAdapterAndReadyState.readyState === "Installed";
        })
        .sort((walletAdapterAndReadyStateA, walletAdapterAndReadyStateB) => {
            return byInstalledStatus(
                walletAdapterAndReadyStateA,
                walletAdapterAndReadyStateB,
            );
        });

    async function handleConnect(wallet: Adapter) {
        $walletStore.select(wallet.name);
        await $walletStore.connect();
    }

    async function copyToClipboard() {
        await navigator.clipboard.writeText($walletStore.publicKey!.toBase58());
    }

    async function handleDisconnect() {
        await $walletStore.disconnect();
    }
</script>

{#if $walletStore.connected}
    <button id="connected-wallet-btn" popovertarget="wallet-menu"
        >Connected/{$walletStore.publicKey}</button
    >
    <div id="wallet-menu" popover>
        <button onclick={copyToClipboard}>Copy Address</button>
        <button onclick={handleDisconnect}>Disconnect</button>
    </div>
{:else}
    <button popovertarget="select-wallet-modal">Connect Solana Wallet</button>
    <div id="select-wallet-modal" popover>
        <ul>
            {#each installedWalletAdaptersWithReadyState as wallet}
                <li>
                    {#if !wallet.adapter.connected}
                        <button
                            onclick={async () => {
                                await handleConnect(wallet.adapter);
                            }}
                            type="button">
                            <img alt="icon of {wallet.adapter.name}" src={wallet.adapter.icon} width="38px"/>
                            {wallet.adapter.name}</button
                        >
                    {:else}
                        <button type="button"
                            >{wallet.adapter.name}/{wallet.adapter
                                .publicKey}</button
                        >
                    {/if}
                </li>
            {/each}
        </ul>
    </div>
{/if}

<style>
    #connected-wallet-btn {
        anchor-name: --connected-wallet-btn;
    }
    #wallet-menu {
        top: anchor(--connected-wallet-btn bottom);
        left: anchor(--connected-wallet-btn left);
    }
</style>
