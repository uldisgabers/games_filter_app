<script lang="ts">
  import { onMount } from "svelte";

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

        getVersions();
      })
      .catch((error) => {
        console.log(error);
      });
  });

  let selectedGame: Game = {
    app_id: "1",
    name: "All",
    icon: "",
  };
  let selectedVersion = "All";
  let selectedCountry = "All";

  const retentionFilterFunc = () => {
    // filteredRetention = retention;
    // countries = [
    //   ...new Set(filteredRetention.map((item) => item.country)),
    // ].sort();

    // if (selectedGame.name === "All") {
    //   countries = [
    //     ...new Set(filteredRetention.map((item) => item.country)),
    //   ].sort();

    //   filteredRetention = retention;
    // } else {
    //   countries = [
    //     ...new Set(filteredRetention.map((item) => item.country)),
    //   ].sort();
    //   filteredRetention = retention.filter((item) => {
    //     return item.app_id === selectedGame.app_id;
    //   });

    //   if (selectedVersion !== "All") {
    //     countries = [
    //       ...new Set(filteredRetention.map((item) => item.country)),
    //     ].sort();

    //     filteredRetention = filteredRetention.filter((item) => {
    //       return item.app_ver === selectedVersion;
    //     });

    //     if (selectedCountry !== "All") {
    //       filteredRetention = filteredRetention.filter((item) => {
    //         return item.country === selectedCountry;
    //       });
    //     }
    //   }

    //   if (selectedCountry !== "All") {
    //     countries = [
    //       ...new Set(filteredRetention.map((item) => item.country)),
    //     ].sort();

    //     filteredRetention = filteredRetention.filter((item) => {
    //       return item.country === selectedCountry;
    //     });
    //   }
    // }

    let filtered = retention.slice(); // Make a shallow copy of the retention array

    if (selectedGame.name !== "All") {
      filtered = filtered.filter((item) => item.app_id === selectedGame.app_id);
    }

    if (selectedVersion !== "All") {
      filtered = filtered.filter((item) => item.app_ver === selectedVersion);
    }

    if (selectedCountry !== "All") {
      filtered = filtered.filter((item) => item.country === selectedCountry);
    }

    // Update countries array after filtering
    countries = [...new Set(filtered.map((item) => item.country))].sort();

    // Update filteredRetention with the final filtered array
    filteredRetention = filtered;
  };

  const handleGameInput = (e: any) => {
    filteredGames = games.filter((game) => {
      return game.name.toLowerCase().includes(e.target.value.toLowerCase());
    });
  };

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

  const handleCountryInput = (e: any) => {
    countries = [
      ...new Set(
        filteredRetention
          .filter((item) =>
            item.country.toLowerCase().includes(e.target.value.toLowerCase()),
          )
          .map((item) => item.country),
      ),
    ];
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

  const showFullNameIfTooLong = (name: string) => {
    if (name.length > 15) {
      return name;
    } else {
      return null;
    }
  };
</script>

<main>
  <!-- ##############---GAME-----######################### -->

  <!-- svelte-ignore a11y-click-events-have-key-events -->
  <!-- svelte-ignore a11y-no-static-element-interactions -->
  <div class="dropdown">
    <div class="dropdown-search" on:click={toggleFilterField}>
      {selectedGame.name}
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
          filteredRetention = retention;
          selectedGame = {
            app_id: "1",
            name: "All",
            icon: "",
          };
          selectedVersion = "All";
          selectedCountry = "All";
          retentionFilterFunc();
          getVersions();
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
            selectedGame = game;
            selectedVersion = "All";
            selectedCountry = "All";

            retentionFilterFunc();
            getVersions();
            // filteredRetention = retention.filter((item) => {
            //   return game.app_id === item.app_id;
            // });
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
          retentionFilterFunc();
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
            retentionFilterFunc();
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
        <input class="filter-input" on:input={handleCountryInput} />
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
          retentionFilterFunc();
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
            retentionFilterFunc();
          }}
          title={showFullNameIfTooLong(country)}
        >
          {#if country.length > 15}
            {country.slice(0, 15) + "..."}
          {:else}
            {country}
          {/if}
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

  <h4>
    Game:{selectedGame.name}<br />Version: {selectedVersion} <br />Country: {selectedCountry}
  </h4>

  <div>
    <table>
      <thead>
        <tr>
          <th>ID</th>
          <th>Version</th>
          <th>Country</th>
        </tr>
      </thead>
      <tbody>
        {#each filteredRetention as gameData}
          <tr>
            <td>{gameData.app_id}</td>
            <td>{gameData.app_ver}</td>
            <td>{gameData.country}</td>
          </tr>
        {/each}
      </tbody>
    </table>
  </div>
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
