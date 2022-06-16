<script>
  import { lastMsgs } from "./Store";
  $: item = getItem();
  let id = 1;
  $: currentBid = 0;
  let isFinished = false;
  const participant = Math.floor(Math.random() * 100) + 1;
  let auctionEnds;
  $: newBid = getMinBid(item, currentBid);

  const getMinBid = (item, currentBid) => {
    return item.then(
      (_item) =>
        (newBid = currentBid
          ? Math.max(_item.min_price, currentBid) + _item.price_step
          : Math.max(_item.min_price, currentBid))
    );
  };

  const makeNewBid = (e) => {
    const data = JSON.stringify({ bid: newBid });
    ws.send(data);
  };

  async function getItem() {
    const response = await fetch(`http://localhost:8000/auction/${id}`);
    const data = await response.json();
    currentBid = data.item.bid;
    return data.item;
  }

  function isJsonString(str) {
    try {
      JSON.parse(str);
    } catch (error) {
      return false;
    }
    return true;
  }

  const ws = new WebSocket(
    `ws://localhost:8000/auction/${id}/ws/${participant}`
  );
  ws.onmessage = (event) => {
    let incomingMessage = event.data;
    if (isJsonString(incomingMessage)) {
      incomingMessage = JSON.parse(incomingMessage);
      if (incomingMessage.new_price) {
        currentBid = incomingMessage.new_price;
        auctionEnds = new Date(incomingMessage.ends);
      }
    } else {
      $lastMsgs = [...$lastMsgs, incomingMessage];
    }
  };
</script>

<main class="grid">
  <div class="container">
    {#await item}
      <p>Loading Info...</p>
    {:then item}
      <p>{item.item_name}</p>
      <p>{item.item_description}</p>
      {#if !currentBid}
        <p>The starting price is ${item.min_price}</p>
      {:else}
        <p>
          {#if !isFinished}
            The current bid price is
          {:else}
            The last bid price was
          {/if} ${currentBid}
        </p>
      {/if}
      <button disabled={isFinished} on:click={makeNewBid}>Bid ${newBid}</button>
    {/await}
  </div>
  <div>
    <p>Welcome, participant #{participant}!</p>
    {#each $lastMsgs as msg}
      <h4>{msg}</h4>
    {/each}
  </div>
</main>

<style>
  .container {
    display: flex;
    justify-self: center;
    flex-direction: column;
    max-height: 100%;
  }
  .grid {
    display: grid;
    gap: 10px;
    grid-template-columns: auto auto;
  }
</style>
