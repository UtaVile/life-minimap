<script>
  import { onMount } from "svelte";
  import Wrld from "wrld.js";

  let container;

  onMount(() => {
    let child = document.createElement("div");
    child.setAttribute("id", "map");
    child.style.height = '500px';
    container.appendChild(child);

    const map = Wrld.map("map", "9c4d9fbedcdddc8634867927eed8dc09");
  });
</script>

<div bind:this={container} />