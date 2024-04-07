<script lang="ts">
  import { onMount } from "svelte";
  import NewMethodFilter from "./components/NewMethodFilter.svelte"

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

  let games: Game[] = [];
  let retention: Retention[] = [];
  let versions: string[] = [];
  let countries: string[] = [];

  let filteredGames: Game[] = [];
  let filteredRetention: Retention[] = [];
  let filteredVersions: string[] = [];

  onMount(async () => {
    fetch("https://storage.googleapis.com/estoty-temp/games.json")
      .then((response) => response.json())
      .then((data) => {
        games = data;
        games.sort((a, b) => a.name.localeCompare(b.name));

        filteredGames = games;
      })
      .catch((error) => {
        console.log(error);
      });

    fetch("https://storage.googleapis.com/estoty-temp/retention.json")
      .then((response) => response.json())
      .then((data) => {
        retention = data;

        filteredRetention = retention;

        countries = [...new Set(retention.map((item) => item.country))].sort(); // get the unique countries data
      
        getVersions()
      })
      .catch((error) => {
        console.log(error);
      });
  });

  const handleGameInput = (e: any) => {
    filteredGames = games.filter((game) => {
      return game.name.toLowerCase().includes(e.target.value.toLowerCase());
    });
  };

  // const getVersions = (retentionArr: Retention[]) => {
  const getVersions = () => {
    versions = [...new Set(filteredRetention.map((item) => item.app_ver))]; // get the unique version data
    versions = versions.sort(
      (a, b) =>
        Number(b.slice(0, b.indexOf("."))) - Number(a.slice(0, a.indexOf("."))),
    ); // sort it decending
    return versions;
  };

  

  const handleVersionInput = (e: any) => {
    versions = [
      ...new Set(
        filteredRetention
          .filter((item) => item.app_ver.includes(e.target.value))
          .map((item) => item.app_ver),
      ),
    ];

    versions.sort(
      (a, b) =>
        Number(b.slice(0, b.indexOf("."))) - Number(a.slice(0, a.indexOf("."))),
    );
  };

  const getDevicesForVersionsCount = (array: Retention[], version: string) => {
    let count = 0;
    array.forEach((item) => {
      if (item.app_ver === version) {
        count += item.days.reduce(
          (accumulator, currentValue) => accumulator + currentValue,
          0,
        );
      }
    });
    return count;
  };

  let isGameSelected = false;

  let selectedGame = "All";
  let selectedVersion = "All";
  let selectedCountry = "All"

  let isGameFilterFieldOpen = false;
  const toggleFilterField = () => {
    isGameFilterFieldOpen = !isGameFilterFieldOpen;
  };

  let isVersionFilterFieldOpen = false;
  const toggleVersionFilterField = () => {
    isVersionFilterFieldOpen = !isVersionFilterFieldOpen;
  };

  let isCountryFilterFieldOpen = false;
  const toggleCountryFilterField = () => {
    isCountryFilterFieldOpen = !isCountryFilterFieldOpen;
  };

</script>

<main>


  <!-- ##############---GAME-----######################### -->

  <!-- svelte-ignore a11y-click-events-have-key-events -->
  <!-- svelte-ignore a11y-no-static-element-interactions -->
  <div class="dropdown">
    <div class="dropdown-search" on:click={toggleFilterField}>
      {selectedGame}
    </div>
    <div
      class="dropdown-search-open"
      style="display: {isGameFilterFieldOpen ? 'block' : 'none'};"
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
          getVersions()
          filteredRetention = retention;
          selectedGame = "All";
          selectedVersion = "All";
          
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
            getVersions()
            
            selectedGame = game.name;
            selectedVersion = "All";

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

  <!-- ##############---VERSION-----######################### -->

  <!-- svelte-ignore a11y-click-events-have-key-events -->
  <!-- svelte-ignore a11y-no-static-element-interactions -->
  <div class="dropdown">
    <div class="dropdown-search" on:click={toggleVersionFilterField}>
      {selectedVersion}
    </div>
    <div
      class="dropdown-search-open"
      style="display: {isVersionFilterFieldOpen ? 'block' : 'none'};"
    >
      <div>
        <input class="filter-input" on:input={handleVersionInput} />
      </div>
      <!-- svelte-ignore a11y-click-events-have-key-events -->
      <!-- svelte-ignore a11y-no-static-element-interactions -->
      <div
        class="dropdown-option"
        on:click={(e) => {
          e.preventDefault();
          toggleVersionFilterField();
          filteredRetention = retention;
          selectedVersion = "All";
        }}
      >
        All
      </div>
      {#each versions as version}
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <!-- svelte-ignore a11y-no-static-element-interactions -->
        <div
          class="dropdown-option"
          on:click={(e) => {
            e.preventDefault();
            selectedVersion = version;
            toggleVersionFilterField();
          }}
        >
          <b>{version}</b> ({getDevicesForVersionsCount(
            filteredRetention,
            version,
          )})
        </div>
      {/each}
    </div>
  </div>

  <!-- ###################-----COUNTRY--------######################### -->

  <!-- svelte-ignore a11y-click-events-have-key-events -->
  <!-- svelte-ignore a11y-no-static-element-interactions -->
  <div class="dropdown">
    <div class="dropdown-search" on:click={toggleCountryFilterField}>
      {selectedCountry}
    </div>
    <div
      class="dropdown-search-open"
      style="display: {isCountryFilterFieldOpen ? 'block' : 'none'};"
    >
      <div>
        <input class="filter-input" on:input={handleVersionInput} />
      </div>
      <!-- svelte-ignore a11y-click-events-have-key-events -->
      <!-- svelte-ignore a11y-no-static-element-interactions -->
      <div
        class="dropdown-option"
        on:click={(e) => {
          e.preventDefault();
          toggleCountryFilterField();
          // filteredRetention = retention;
          selectedCountry = "All";
        }}
      >
        All
      </div>
      {#each countries as country}
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <!-- svelte-ignore a11y-no-static-element-interactions -->
        <div
          class="dropdown-option"
          on:click={(e) => {
            e.preventDefault();
            selectedCountry = country;
            toggleCountryFilterField();
          }}
        >
          {country}
        </div>
      {/each}
    </div>
  </div>

  <!-- <DropdownMenu
    {retention}
    {games}
    {filteredGames}
    {filteredRetention}
    bind:selectedGame={selectedGame}
  /> -->

  <h4>Game:{selectedGame}<br />Version: {selectedVersion}</h4>


<NewMethodFilter />

</main>

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

  .dropbtn {
    background-color: #4caf50;
    color: white;
    min-width: 200px;
    padding: 16px;
    font-size: 16px;
    border: none;
    cursor: pointer;
  }

  .dropdown {
    position: relative;
    display: inline-block;
  }

  .dropdown-content {
    display: none;
    position: absolute;
    background-color: #f9f9f9;
    min-width: 200px;
    box-shadow: 0px 8px 16px 0px rgba(0, 0, 0, 0.2);
    z-index: 1;
  }

  .dropdown-content a {
    color: black;
    padding: 12px 16px;
    text-decoration: none;
    display: block;
  }

  .dropdown-content a:hover {
    background-color: #f1f1f1;
  }

  .dropdown:hover .dropdown-content {
    display: block;
  }

  .dropdown:hover .dropbtn {
    background-color: #3e8e41;
  }

  ::placeholder {
    color: #fff;
    opacity: 1; /* Firefox */
  }
</style>