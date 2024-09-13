<script lang="ts">
    import { walletStore } from "./walletStore";
    import {
        SystemProgram,
        PublicKey,
        LAMPORTS_PER_SOL,
        TransactionMessage,
        clusterApiUrl,
        Connection,
        VersionedTransaction,
    } from "@solana/web3.js";
    const receiverAddress = "FDjn87xPsLiXwakFygi4uEdet568o7A22UboxrUCwu7A";

    let explorerUrl = "";

    async function simpleTransfer(amountInSol: number) {
        const lamports = amountInSol * LAMPORTS_PER_SOL;
        const transferToTestWalletIx = SystemProgram.transfer({
            lamports: lamports,
            // `fromPubkey` - from MUST sign the transaction
            fromPubkey: $walletStore.publicKey!,
            // `toPubkey` - does NOT have to sign the transaction
            toPubkey: new PublicKey(receiverAddress),
            programId: SystemProgram.programId,
        });

        const connection = new Connection(clusterApiUrl("devnet"), "confirmed");
        let recentBlockhash = await connection
            .getLatestBlockhash()
            .then((res) => res.blockhash);

        // create a transaction message
        const message = new TransactionMessage({
            payerKey: $walletStore.publicKey!,
            recentBlockhash,
            instructions: [transferToTestWalletIx],
        }).compileToV0Message();

        const tx = new VersionedTransaction(message);

        // console.log("tx before signing:", tx);

        // sign the transaction with our needed Signers (e.g. `payer` and `keypair`)
        const sig = await $walletStore.adapter!.sendTransaction(tx, connection);

        explorerUrl = `https://explorer.solana.com/tx/${sig?.toString()}?cluster=devnet`;
    }
</script>

{#if $walletStore.connected}
    <p>Click to transfer 0.1 SOL to {receiverAddress} on devnet</p>
    <button onclick="{async () => await simpleTransfer(0.1)}">transfer</button>
    {#if explorerUrl}
        <a href="{explorerUrl}" target="_blank">View on Solana Explorer</a>
    {/if}
{:else}
    <p>Please connect wallet first to do transfer!</p>
{/if}
