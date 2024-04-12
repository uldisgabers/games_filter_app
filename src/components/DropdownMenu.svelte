<script lang="ts">
  export let retention: Retention[];
  export let filteredRetention: Retention[];
  export let filteredGames: Game[];
  export let games: Game[];
  export let selectedGame: Game;

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

  let isGameFilterFieldOpen = false;
  const toggleFilterField = () => {
    isGameFilterFieldOpen = !isGameFilterFieldOpen;
  };
</script>

<div class="dropdown">
  <!-- svelte-ignore a11y-interactive-supports-focus -->
  <div
    class="dropdown-search"
    tabIndex="0"
    on:click={toggleFilterField}
    role="button"
    on:keydown={toggleFilterField}
  >
    {selectedGame.name}
  </div>
  <div
    class="dropdown-search-open"
    style="display: {isGameFilterFieldOpen ? 'block' : 'none'};"
  >
    <div>
      <input class="filter-input" on:input={handleGameInput} />
    </div>

    <div
      class="dropdown-option"
      tabindex="0"
      on:click={(e) => {
        e.preventDefault();

        toggleFilterField();
        filteredRetention = retention;
        selectedGame = {
          app_id: "1",
          name: "All",
          icon: "",
        };
      }}
      role="button"
      on:keydown={(e) => {
        e.preventDefault();
        toggleFilterField();
      }}
    >
      All
    </div>
    {#each filteredGames as game}
      <div
        class="dropdown-option"
        tabindex="0"
        on:click={(e) => {
          e.preventDefault();
          toggleFilterField();
          selectedGame = game;
        }}
        role="button"
        on:keydown={(e) => {
          e.preventDefault();
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
    min-width: 300px;
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
    z-index: 10;
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

  .dropdown {
    position: relative;
    display: inline-block;
  }

  ::placeholder {
    color: #fff;
    opacity: 1;
  }
</style>
