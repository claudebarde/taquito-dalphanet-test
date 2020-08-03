<script>
  import { onMount } from "svelte";
  import { TezosToolkit } from "@taquito/taquito";
  import { validateAddress } from "@taquito/utils";
  import { pkh as publicKey } from "../../faucet.json";

  let implicitAccountBalance,
    address,
    loadingAddress,
    loadingContract,
    contractAddress,
    contractBalance,
    storage,
    errorAddress,
    errorContract;
  let Tezos;

  const contractExample = "KT1FgLVdSuTWnY1pzJ1bi5gCTbTPiTZHiKJa";

  const findBalance = async () => {
    if (validateAddress(address) === 3) {
      implicitAccountBalance = undefined;
      errorAddress = false;
      loadingAddress = true;
      try {
        const balance = await Tezos.tz.getBalance(address);
        if (balance) {
          implicitAccountBalance = balance;
        } else {
          throw new Error("No balance");
        }
      } catch (error) {
        console.log(error);
        errorAddress = true;
      } finally {
        loadingAddress = false;
      }
    }
  };

  const loadContract = async () => {
    if (validateAddress(contractAddress) === 3) {
      storage = undefined;
      errorContract = false;
      loadingContract = true;
      try {
        contractBalance = await Tezos.tz.getBalance(contractAddress);
        const contract = await Tezos.wallet.at(contractAddress);
        storage = await contract.storage();
      } catch (error) {
        console.log(error);
        errorContract = true;
        if (
          error &&
          error.data &&
          error.data.prim &&
          error.data.prim === "sapling_state"
        ) {
          storage = "Sapling state";
        }
      } finally {
        loadingContract = false;
      }
    }
  };

  onMount(async () => {
    Tezos = new TezosToolkit();
    Tezos.setProvider({ rpc: "https://dalphanet.smartpy.io" });
    address = publicKey;
    contractAddress = contractExample;
  });
</script>

<style>
  .main-box {
    width: 50%;
    margin: 0 auto;
  }

  label {
    text-align: left;
  }
</style>

<section class="hero is-fullheight">
  <div class="hero-body has-background-primary-light">
    <div class="container">
      <div class="box main-box">
        <h1 class="title has-text-centered">Taquito Test on Dalphanet</h1>
        <h3 class="subtitle is-4 has-text-centered">Implicit accounts</h3>
        <label class="has-text-left">
          <span>Find the balance of an address:</span>
          <div class="field has-addons">
            <div class="control" style="width:100%">
              <input
                class="input"
                class-is-danger={errorAddress}
                type="text"
                placeholder="Find account balance"
                bind:value={address}
                disabled={loadingAddress} />
            </div>
            <div class="control">
              <button
                class="button is-info"
                class:is-loading={loadingAddress}
                on:click={findBalance}>
                Search
              </button>
            </div>
          </div>
        </label>
        <p>
          {#if implicitAccountBalance}
            XTZ {(implicitAccountBalance.toNumber() / 1000000).toLocaleString('en-US')}
          {:else}&nbsp;{/if}
        </p>
        <h3 class="subtitle is-4 has-text-centered">Smart contracts</h3>
        <label class="has-text-left">
          <span>Find the storage of a contract:</span>
          <div class="field has-addons">
            <div class="control" style="width:100%">
              <input
                class="input"
                class:is-danger={errorContract}
                type="text"
                placeholder="Find contract storage"
                bind:value={contractAddress}
                disabled={loadingContract} />
            </div>
            <div class="control">
              <button
                class="button is-info"
                class:is-loading={loadingContract}
                on:click={loadContract}>
                Search
              </button>
            </div>
          </div>
        </label>
        <p>
          {#if contractBalance}
            XTZ {(contractBalance.toNumber() / 1000000).toLocaleString('en-US')}
          {:else}&nbsp;{/if}
        </p>
        <p>
          {#if storage}Storage: {JSON.stringify(storage)}{/if}
        </p>
      </div>
    </div>
  </div>
</section>
