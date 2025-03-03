<style>

#scrollTopBtn {
  position: fixed;
  bottom: 70px; /* Poziționat deasupra barei de căutare */
  right: 20px;
  background: #3a87da;
  color: white;
  border: none;
  border-radius: 80%;
  border: none;
  width: 45px;
  height: 45px;
  font-size: 24px;
  font-weight: bold;
  cursor: pointer;
  display: none; /* Ascuns inițial */
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  transition: opacity 0.3s, transform 0.3s;

  display: flex;
  justify-content: center;
  align-items: center;
}

#scrollTopBtn:hover {
  background: #0056b3;
}

#scrollTopBtn.show {
  display: block;
  display: flex; /* Folosim flexbox pentru centrare */
  opacity: 1;
}

#scrollTopBtn {
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.3s ease-in-out;
}

#scrollTopBtn.show {
  opacity: 1;
  pointer-events: auto;
}

 /* Fixează bara de căutare jos */
.search-bar {
  position: fixed;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 95%;
  max-width: 500px;
  padding: 12px;
  background: white;
  border-top: 1px solid #ccc;
  box-shadow: 0 -2px 5px rgba(0, 0, 0, 0.1);
}
 
 /* Bara de căutare */
 input {
   display: block;
   width: 95%;
   max-width: 500px;
   padding: 12px;
   font-size: 1rem;
   border-radius: 8px;
   border: 1px solid #ccc;
   transition: border 0.3s;
 }
 
 input:focus {
   border-color: #007bff;
   outline: none;
 }
 
 /* Lista produse */
 ul {
   list-style-type: none;
   padding: 0;
   margin: 20px auto;
   width: 95%;
   max-width: 600px;
 }
 
 li {
   background: white;
   padding: 15px;
   margin: 10px 0;
   border-radius: 8px;
   box-shadow: 2px 2px 10px rgba(0, 0, 0, 0.1);
   transition: 0.3s;
   cursor: pointer;
 }
 
 li:hover {
   box-shadow: 2px 2px 15px rgba(0, 0, 0, 0.2);
 }
 
 /* Informații suplimentare */
 .extra-info {
   display: none;
   margin-top: 10px;
   font-size: 0.9rem;
   color: #666;
 }
 
 .expanded .extra-info {
   display: block;
 }
 
 /* Responsive - adaptare pentru telefoane */
 @media (max-width: 600px) {
  input {
     width: 90%;
     font-size: 0.95rem;
     color: black !important;
   }
 
   li {
     font-size: 1rem;
     color: black !important;
     text-align: center;
     padding: 12px;
   }
 
   .extra-info {
     font-size: 0.85rem;
     color: black !important;
   }
 }

 @media (prefers-color-scheme: black) {
  .extra-info {
    color: white !important;
  }
}
 
 </style>


<script>
  import { onMount } from "svelte";

  let produse = [];
  let searchTerm = "";
  let expanded = {};
  let lastUpdate = 0;
  let searchTimeout;
  let showScrollBtn = false;

  function scrollToTop() {
  window.scrollTo({ top: 0, behavior: "smooth" });
}

function handleScroll() {
  showScrollBtn = window.scrollY > 300; // Apare după ce ai scrollat 300px
}

onMount(() => {
  window.addEventListener("scroll", handleScroll);
  return () => window.removeEventListener("scroll", handleScroll);
});

  async function fetchLastUpdate() {
    try {
      const response = await fetch("https://glasssvelte-backend.onrender.com/api/products/last-update");
      const data = await response.json();
      return data.timestamp;
    } catch (error) {
      console.error("Eroare la verificarea actualizărilor:", error);
      return lastUpdate;
    }
  }

  async function fetchProduse(forceUpdate = false) {
    try {
      let cachedData = localStorage.getItem("produse");
      let cachedTimestamp = Number(localStorage.getItem("lastUpdate")) || 0;

      if (cachedData && !forceUpdate) {
        produse = JSON.parse(cachedData);
        lastUpdate = cachedTimestamp;
      }

      const newUpdate = await fetchLastUpdate();
      if (newUpdate <= lastUpdate && !forceUpdate) return;

      console.log("🔄 Date noi găsite, actualizăm produsele...");
      const url = searchTerm
        ? `https://glasssvelte-backend.onrender.com/api/products?search=${encodeURIComponent(searchTerm)}`
        : "https://glasssvelte-backend.onrender.com/api/products";

      const response = await fetch(url);
      const data = await response.json();

      let newProduse = data.map(produs => ({
        ...produs,
        expanded: expanded[produs.pa_id] || false
      }));

      produse = newProduse;
      localStorage.setItem("produse", JSON.stringify(produse));
      localStorage.setItem("lastUpdate", newUpdate);
      lastUpdate = newUpdate;
    } catch (error) {
      console.error("Eroare la preluarea produselor:", error);
    }
  }

  function toggleExpand(id) {
    expanded = { ...expanded, [id]: !expanded[id] };
  }

//   function handleSearch() {
//   clearTimeout(searchTimeout);
//   searchTimeout = setTimeout(() => {
//     produse = JSON.parse(localStorage.getItem("produse") || "[]")
//       .filter(produs => produs.a_marca_model.toLowerCase().includes(searchTerm.toLowerCase()));
//   }, 300);
// }

function handleSearch() {
    clearTimeout(searchTimeout);
    searchTimeout = setTimeout(() => {
      produse = JSON.parse(localStorage.getItem("produse") || "[]").filter(produs => 
        produs.a_marca_model.toLowerCase().includes(searchTerm.toLowerCase()) ||
        produs.cod.toLowerCase().includes(searchTerm.toLowerCase())
      );
    }, 300);
  }

  function clearSearch() {
    searchTerm = "";
    handleSearch();
  }

  onMount(fetchProduse);

  setInterval(fetchProduse, 80000);
</script>

<input 
  type="text" 
  class="search-bar" 
  bind:value={searchTerm} 
  placeholder="🔍 Caută produs..." 
  on:input={handleSearch} 
  on:click={clearSearch} 
/>
{#if produse.length > 0}
  <ul>
    {#each produse as produs}
      <!-- <li class:expanded={expanded[produs.pa_id]} on:click={() => toggleExpand(produs.pa_id)}> -->
        <li 
        class:expanded={expanded[produs.pa_id]} 
        on:click={() => toggleExpand(produs.pa_id)}
        on:keydown={(e) => { if (e.key === "Enter" || e.key === " ") toggleExpand(produs.pa_id); }}
        tabindex="0"
      >
      
        <strong>{produs.a_marca_model}</strong> - {produs.cod}

        {#if expanded[produs.pa_id]}
          <div class="extra-info">
            <p>Celula: {produs.nume_celula}</p>
            <p>Stoc: {produs.p_count}</p>
            <p>Preț: {produs.p_price} MDL</p>
            <p>{produs.is_updated}</p>
          </div>
        {/if}
      </li>

      <!-- <li class:expanded={expanded[produs.pa_id]}>
        <button 
          on:click={() => toggleExpand(produs.pa_id)}
        >
          <strong>{produs.a_marca_model}</strong> - {produs.cod}
        </button>
      
        {#if expanded[produs.pa_id]}
          <div class="extra-info">
            <p>Celula: {produs.nume_celula}</p>
            <p>Stoc: {produs.p_count}</p>
            <p>Preț: {produs.p_price} MDL</p>
            <p>{produs.is_updated}</p>
          </div>
        {/if}
      </li> -->
      
    {/each}
  </ul>
  <button id="scrollTopBtn" class:show={showScrollBtn} on:click={scrollToTop}>↑</button>

{:else}
  <p style="text-align:center;">⏳ Se încarcă produsele...</p>
{/if}