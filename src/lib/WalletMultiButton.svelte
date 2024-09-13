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

    function abbrAddress(address: string) {
        return `${address.slice(0, 4)}...${address.slice(-4)}`;
    }
</script>

{#if $walletStore.connected}
    <button id="connected-wallet-btn" popovertarget="connected-wallet-menu"
        >{abbrAddress($walletStore.publicKey!.toBase58())}</button
    >
    <div id="connected-wallet-menu" popover="auto">
        <ul>
            <li><button onclick={copyToClipboard}>Copy Address</button></li>
            <li><button onclick={handleDisconnect}>Disconnect</button></li>
        </ul>
    </div>
{:else}
    <button id="select-wallet-btn" popovertarget="select-wallet-modal"
        >Connect Solana Wallet</button
    >
    <div id="select-wallet-modal" popover="auto">
        <ul>
            {#each installedWalletAdaptersWithReadyState as wallet}
                <li>
                    {#if !wallet.adapter.connected}
                        <button
                            onclick={async () => {
                                await handleConnect(wallet.adapter);
                            }}
                            type="button"
                        >
                            <img
                                alt="icon of {wallet.adapter.name}"
                                src={wallet.adapter.icon}
                                width="38px"
                            />
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
    ul {
        list-style-type: none;
        padding: 0;
        margin: 0;
    }
    li {
        list-style-type: none;
    }
    [popover] {
        margin: 0;
        padding: 0;
        border: 0;
    }

    #connected-wallet-btn {
        anchor-name: --connected-wallet-btn;
    }
    #connected-wallet-menu {
        position: absolute;
        position-anchor: --connected-wallet-btn;
        right: anchor(right);
        top: anchor(bottom);
        inset-area: bottom;
    }

    #select-wallet-btn {
        anchor-name: --select-wallet-btn;
    }
    #select-wallet-modal {
        position-anchor: --select-wallet-btn;
        right: anchor(right);
        bottom: anchor(bottom);
        inset-area: bottom;
    }
</style>
