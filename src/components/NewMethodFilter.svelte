<script lang="ts">
  import { onMount } from "svelte";
  import { Bar } from "svelte-chartjs";
  import DropdownMenu from "./DropdownMenu.svelte"

  import {
    Chart,
    Title,
    Tooltip,
    Legend,
    BarElement,
    CategoryScale,
    LinearScale,
  } from "chart.js";

  Chart.register(
    Title,
    Tooltip,
    Legend,
    BarElement,
    CategoryScale,
    LinearScale,
  );

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

  type CountriesWithDevices = {
    name: string;
    devices: number;
  };

  let games: Game[] = [];
  let retention: Retention[] = [];
  let versions: string[] = [];
  let countries: string[] = [];

  let filteredGames: Game[] = [];
  let filteredRetention: Retention[] = [];
  let filteredVersions: string[] = [];
  let countriesWithDevices: CountriesWithDevices[] = [];

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
        // getCountries(retention);
        getCountriesWithDeviceCounts(retention);
      })
      .catch((error) => {
        console.log(error);
      });
  });

  let selectedGame: Game = {
    app_id: "1",
    name: "All",
    icon: "", // default selection of All
  };
  let selectedVersion = "All";
  let selectedCountry = "All";

  const retentionFilterFunc = () => {
    let filtered = retention.slice();

    if (selectedGame.name !== "All") {
      filtered = filtered.filter((item) => item.app_id === selectedGame.app_id);
    }

    if (selectedVersion !== "All") {
      filtered = filtered.filter((item) => item.app_ver === selectedVersion);
    }

    if (selectedCountry !== "All") {
      filtered = filtered.filter((item) => item.country === selectedCountry);
    }

    filteredRetention = filtered;
  };

  const handleGameInput = (e: any) => {
    filteredGames = games.filter((game) => {
      return game.name.toLowerCase().includes(e.target.value.toLowerCase());
    });
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
    getCountriesWithDeviceCounts(retention);

    countriesWithDevices = [
      ...new Set(
        countriesWithDevices
          .filter((item) =>
            item.name.toLowerCase().includes(e.target.value.toLowerCase()),
          )
          .map((item) => ({
            name: item.name,
            devices: item.devices,
          })),
      ),
    ];
  };

  // const getVersions = () => {
  //   versions = [...new Set(filteredRetention.map((item) => item.app_ver))]; // get the unique version data
  //   versions = versions.sort(
  //     (a, b) =>
  //       Number(b.slice(0, b.indexOf("."))) - Number(a.slice(0, a.indexOf("."))),
  //   ); // sort it decending
  //   return versions;
  // };

  const getVersions = () => {
    let filteredVersions: string[] = [];

    retention.forEach((item) => {
      if (
        (selectedGame.app_id === item.app_id || selectedGame.app_id === "1") &&
        (selectedCountry === item.country || selectedCountry === "All") &&
        filteredVersions.indexOf(item.app_ver) === -1
      ) {
        filteredVersions.push(item.app_ver);
      }
    });

    versions = filteredVersions.sort(
      (a, b) =>
        Number(b.slice(0, b.indexOf("."))) - Number(a.slice(0, a.indexOf("."))),
    );
  };

  const getCountries = (array: Retention[]) => {
    let filteredCountries: string[] = [];

    array.forEach((item) => {
      if (
        (selectedGame.app_id === item.app_id || selectedGame.app_id === "1") &&
        (selectedVersion === item.app_ver || selectedVersion === "All") &&
        filteredCountries.indexOf(item.country) === -1
      ) {
        filteredCountries.push(item.country);
      }
    });

    countries = filteredCountries;
  };

  const getDevicesForVersionsCount = (
    array: Retention[],
    game: string,
    version: string,
    country: string,
  ) => {
    let count = 0;
    array.forEach((item) => {
      if (
        (item.app_ver === version || version === "All") &&
        (item.app_id === game || game === "1") &&
        (item.country === country || country === "All")
      ) {
        count += item.days[0];
        return;
      }
    });

    return count;
  };

  const getDevicesForCountryCount = (
    array: Retention[],
    game: string,
    version: string,
    country: string,
  ) => {
    let count = 0;
    array.forEach((item) => {
      if (
        (item.app_ver === version || version === "All") &&
        (item.app_id === game || game === "1") &&
        (item.country === country || country === "All")
      ) {
        count += item.days[0];
      }
    });
    return count;
  };

  const getCountriesWithDeviceCounts = (array: Retention[]) => {
    let filteredCountries: string[] = [];

    array.forEach((item) => {
      if (
        (selectedGame.app_id === item.app_id || selectedGame.app_id === "1") &&
        (selectedVersion === item.app_ver || selectedVersion === "All") &&
        filteredCountries.indexOf(item.country) === -1
      ) {
        filteredCountries.push(item.country);
      }
    });

    const countriesWithCounts = filteredCountries.map((country) => ({
      name: country,
      devices: getDevicesForCountryCount(
        array,
        selectedGame.app_id,
        selectedVersion,
        country,
      ),
    }));

    countriesWithDevices = countriesWithCounts.sort((a, b) => {
      return b.devices - a.devices;
    });
  };

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

  let showTable = false;
  let showChart = false;

  const data = {
    labels: ["D0", "D5", "D10", "D20", "D25", "D30", "D60", "D90"],
    datasets: [
      {
        label: "",
        data: [100, 19, 3, 5, 2, 3, 1, 1],
        backgroundColor: ["rgba(98,  182, 239,0.4)"],
        borderWidth: 2,
        borderColor: ["rgba(98,  182, 239, 1)"],
      },
    ],
  };

  const calculateChartData = (filteredRetention: Retention[]) => {
    const days = [0, 5, 10, 20, 25, 30, 60, 90];

    data.datasets[0].label = `% of retention - app version: ${filteredRetention[0].app_ver} in ${filteredRetention[0].country}`;

    data.datasets[0].data = days.map((item) => {
      return Number(
        (
          (filteredRetention[0].days[item] / filteredRetention[0].days[0]) *
          100
        ).toFixed(0),
      );
    });
  };
</script>

<main>
  <!-- ##############---GAME-----######################### -->

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
          selectedVersion = "All";
          selectedCountry = "All";
          retentionFilterFunc();
          getVersions();
          getCountries(retention);
          getCountriesWithDeviceCounts(retention);
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
            selectedVersion = "All";
            selectedCountry = "All";

            retentionFilterFunc();
            getVersions();
            getCountries(retention);
            getCountriesWithDeviceCounts(retention);
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
          getCountries(retention);
          getCountriesWithDeviceCounts(retention);
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
            getCountries(retention);
            getCountriesWithDeviceCounts(retention);
          }}
        >
          <b>{version}</b> ({getDevicesForVersionsCount(
            retention,
            selectedGame.app_id,
            version,
            selectedCountry,
          )})
        </div>
      {/each}
    </div>
  </div>

  <!-- ###################-----COUNTRY--------######################### -->

  <!-- svelte-ignore a11y-click-events-have-key-events -->
  <!-- svelte-ignore a11y-no-static-element-interactions -->
  <div class="dropdown">
    <div
      class="dropdown-search"
      on:click={toggleCountryFilterField}
      title={showFullNameIfTooLong(selectedCountry)}
    >
      {#if selectedCountry.length > 15}
        {selectedCountry.slice(0, 15) + "..."}
      {:else}
        {selectedCountry}
      {/if}
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
          getVersions();
        }}
      >
        All
      </div>
      {#each countriesWithDevices as country}
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <!-- svelte-ignore a11y-no-static-element-interactions -->
        <div
          class="dropdown-option"
          on:click={(e) => {
            e.preventDefault();
            selectedCountry = country.name;
            toggleCountryFilterField();
            retentionFilterFunc();
            getVersions();
          }}
          title={showFullNameIfTooLong(country.name)}
        >
          {#if country.name.length > 15}
            {country.name.slice(0, 15) + "..."} ({country.devices})
          {:else}
            {country.name} ({country.devices})
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

  <!-- <h4>
    Game:{selectedGame.name}<br />Version: {selectedVersion} <br />Country: {selectedCountry}
  </h4> -->
  <button
    on:click={() => {
      showTable = true;
      showChart = false;
    }}>Table</button
  >
  <button
    on:click={() => {
      calculateChartData(filteredRetention);
      showTable = false;
      showChart = true;
    }}>Chart</button
  >

  {#if showTable}
    <div style="overflow: auto; height:800px; margin-top: 1rem;">
      <table>
        <thead>
          <tr class="table-col-names">
            <th class="sticky-col first-col">Version</th>
            <th class="sticky-col second-col">Country</th>
            {#each { length: 91 } as _, i}
              <th>D{i + 1 - 1}</th>
            {/each}
          </tr>
        </thead>
        <tbody>
          {#each filteredRetention as gameData}
            <tr>
              <td class="sticky-col first-col">{gameData.app_ver}</td>
              <td class="sticky-col second-col">{gameData.country}</td>
              {#each gameData.days as day, i}
                <td>{((day / gameData.days[0]) * 100).toFixed(0)}%</td>
              {/each}
            </tr>
          {/each}
        </tbody>
      </table>
    </div>
  {:else if showChart}
    {#if filteredRetention.length !== 1}
      <h3>Select values for all options to see the chart!</h3>
    {:else}
      <Bar {data} options={{ responsive: true }} />
    {/if}
  {/if}
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
    opacity: 1; /* Firefox */
  }

  .sticky-col {
    position: sticky;
    background-color: rgb(224, 224, 224);
  }

  .first-col {
    min-width: 100px;
    left: 0px;
  }

  .second-col {
    min-width: 150px;
    left: 100px;
  }

  table {
    border-collapse: separate;
    border-spacing: 0;
  }

  table tbody {
    height: 100px;
  }

  table th {
    text-align: center;
    position: sticky;
    top: 0;
    z-index: 2;
  }

  table th:nth-child(1) {
    left: 0;
    z-index: 3;
  }

  table th:nth-child(2) {
    left: 100;
    z-index: 3;
  }

  table td {
    text-align: center;
    white-space: pre;
  }

  table tbody tr td:nth-child(1) {
    position: sticky;
    left: 0;
    z-index: 1;
  }

  table tbody tr td:nth-child(2) {
    /* position: sticky; */
    /* left: 100; */
    z-index: 1;
  }

  table td {
    padding: 0.5rem;
    border: 1px solid rgb(180, 180, 180);
  }
  table th {
    padding: 0.5rem;
    border: 1px solid rgb(180, 180, 180);
  }

  th {
    position: sticky;
    top: 0;
    background-color: rgb(230, 230, 230);
  }
</style>
