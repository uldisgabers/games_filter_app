<script lang="ts">
  export let retention: Retention[];
  export let filteredRetention: Retention[]
  export let filteredGames: Game[]
  export let games: Game[]
  export let selectedGame: Game

  type Game = {
    app_id: string;
    name: string;
    icon: string;
  };

  type Retention = {
    app_id: string;
    app_ver: string;
    country: string;
    days: number[];
  };


  const handleGameInput = (e: any) => {
    filteredGames = games.filter((game) => {
      return game.name.toLowerCase().includes(e.target.value.toLowerCase());
    });
  };

  let isFilterFieldOpen = false;
  const toggleFilterField = () => {
    isFilterFieldOpen = !isFilterFieldOpen;
  };
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-static-element-interactions -->
<div>
  <div class="dropdown-search" on:click={toggleFilterField}>{selectedGame}</div>
  <div
    class="dropdown-search-open"
    style="display: {isFilterFieldOpen ? 'block' : 'none'};"
  >
    <div>
      <input class="filter-input" on:input={handleGameInput} />
    </div>
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <!-- svelte-ignore a11y-no-static-element-interactions -->
    <div
      class="dropdown-option"
      on:click={(e) => {
        e.preventDefault();
        toggleFilterField();
        filteredRetention = retention;
        selectedGame = "All";
  
      }}
    >
      All
    </div>
    {#each filteredGames as game}
      <!-- svelte-ignore a11y-click-events-have-key-events -->
      <!-- svelte-ignore a11y-no-static-element-interactions -->
      <div
        class="dropdown-option"
        on:click={(e) => {
          e.preventDefault();
          toggleFilterField();
          selectedGame = game.name;
         

          filteredRetention = retention.filter((item) => {
            return game.app_id === item.app_id;
          });
        }}
      >
        <img src={game.icon} width="20" height="20" alt={game.name} />
        {game.name}
      </div>
    {/each}
  </div>
</div>

<style>
  .dropdown-search {
    max-width: 300px;
    width: 100%;
    background-color: rgb(143, 201, 255);
    color: #fff;
    padding: 16px;
    font-size: 16px;
    border: none;
    cursor: pointer;
    margin-top: 1rem;
  }

  .dropdown-search:hover {
    background-color: rgb(108, 180, 247);
  }

  .dropdown-search-open {
    max-width: 300px;
    width: 100%;
    color: #000000;
    background-color: rgb(179, 217, 253);
    padding: 16px;
    font-size: 16px;
    border: none;
    cursor: pointer;
    position: absolute;
  }

  .filter-input {
    padding: 0.5rem;
    border: none;
    border-radius: 6px;
    width: 95%;
  }

  .dropdown-option {
    padding: 0.5rem;
  }

  .dropdown-option:hover {
    background-color: rgb(145, 199, 250);
  }
</style>
