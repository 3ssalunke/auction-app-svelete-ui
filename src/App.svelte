<script>
  import {Router, Route, Link} from "svelte-navigator"
  import Auction from "./Auction.svelte";
</script>

<main>
  <Router>
    <Route path="/">
      <h1>Auction</h1>
      <Link to="auction/1">Auc 1</Link>
    </Route>
    <Route path="/auction/:id">
      <Auction></Auction>
    </Route>
  </Router>
</main>

<style>
  
</style>
