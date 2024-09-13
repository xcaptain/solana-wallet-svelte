<script lang="ts">
    import type { Adapter } from "@solana/wallet-adapter-base";
    import { initialize } from "./walletStore";
    import type { WalletError } from "@solana/wallet-adapter-base";

    export let localStorageKey: string;
    export let wallets: Adapter[];
    export let autoConnect = false;
    export let onError = (error: WalletError) => console.error(error);

    $: wallets && initialize({ wallets, autoConnect, localStorageKey, onError });
</script>
