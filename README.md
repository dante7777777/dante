// the circuit must be known at compile time.
const samples = 8;
// Number of blocks between each sample.
const spacing = 900;
// Validate that the block number is greater than the number of samples times the spacing
if (inputs.blockNumber.value() <= (samples * spacing)) {
  throw new Error("Block number must be greater than the number of samples times the spacing");
}

// Perform the block number validation in the circuit as well
checkLessThan(mul(samples, spacing), inputs.blockNumber);

// Get account balance at the sample block numbers
let sampledAccounts = new Array(samples);
for (let i = 0; i < samples; i++) {
  const sampleBlockNumber: CircuitValue = sub(inputs.blockNumber, mul(spacing, i));
  const account = getAccount(sampleBlockNumber, inputs.address);
  sampledAccounts[i] = account;
}

// Accumulate all of the balances to `total`
let total = constant(0);
for (const account of sampledAccounts) {
  const balance: CircuitValue256 = await account.balance();
  total = add(total, balance.toCircuitValue());
}

// Divide the total amount by the number of samples to get the average value
const average = div(total, samples);

// We call `addToCallback` on all values that we would like to be passed to our contract after the circuit has
// been proven in ZK. The values can then be handled by our contract once the prover calls the callback function.
addToCallback(inputs.blockNumber);
addToCallback(inputs.address);
addToCallback(average);
