<script lang="ts">
  import { onMount } from "svelte";
  import { Bar } from "svelte-chartjs";
  import DropdownMenu from "./components/DropdownMenu.svelte";
  import Table from "./components/Table.svelte";

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

  let filteredGames: Game[] = [];
  let filteredRetention: Retention[] = [];
  let countriesWithDevices: CountriesWithDevices[] = [];

  let selectedGame: Game = {
    app_id: "1",
    name: "All",
    icon: "", // default selection of All
  };
  let selectedVersion = "All";
  let selectedCountry = "All";

  let showTable = false;
  let showChart = false;

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

        getVersions();

        getCountriesWithDeviceCounts(retention);
      })
      .catch((error) => {
        console.log(error);
      });
  });

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

    console.log(countriesWithDevices);
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

  $: onGameChange(selectedGame);

  const onGameChange = (selectedGame: Game) => {
    selectedVersion = "All";
    selectedCountry = "All";
    retentionFilterFunc();
    getVersions();
    getCountriesWithDeviceCounts(retention);
  };
</script>

<main>
  <DropdownMenu
    {filteredGames}
    {filteredRetention}
    {games}
    {retention}
    bind:selectedGame
  />

  <div class="dropdown">
    <div
      class="dropdown-search"
      on:click={toggleVersionFilterField}
      on:keydown={toggleVersionFilterField}
      tabindex="0"
      role="button"
    >
      {selectedVersion}
    </div>
    <div
      class="dropdown-search-open"
      style="display: {isVersionFilterFieldOpen ? 'block' : 'none'};"
    >
      <div>
        <input class="filter-input" on:input={handleVersionInput} />
      </div>

      <div
        class="dropdown-option"
        tabindex="0"
        on:click={(e) => {
          e.preventDefault();
          toggleVersionFilterField();
          filteredRetention = retention;
          selectedVersion = "All";
          retentionFilterFunc();
          getCountriesWithDeviceCounts(retention);
        }}
        role="button"
        on:keydown={(e) => {
          e.preventDefault();
        }}
      >
        All
      </div>
      {#each versions as version}
        <div
          class="dropdown-option"
          tabindex="0"
          on:click={(e) => {
            e.preventDefault();
            selectedVersion = version;
            toggleVersionFilterField();
            retentionFilterFunc();
            getCountriesWithDeviceCounts(retention);
            calculateChartData(filteredRetention);
          }}
          role="button"
          on:keydown={(e) => {
            e.preventDefault();
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

  <div class="dropdown">
    <div
      class="dropdown-search"
      tabindex="0"
      on:click={toggleCountryFilterField}
      role="button"
      on:keydown={(e) => {
        e.preventDefault();
      }}
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

      <div
        class="dropdown-option"
        tabindex="0"
        on:click={(e) => {
          e.preventDefault();
          toggleCountryFilterField();
          selectedCountry = "All";
          retentionFilterFunc();
          getVersions();
        }}
        role="button"
        on:keydown={(e) => {
          e.preventDefault();
        }}
      >
        All
      </div>
      {#each countriesWithDevices as country}
        <div
          class="dropdown-option"
          tabindex="0"
          on:click={(e) => {
            e.preventDefault();
            selectedCountry = country.name;
            toggleCountryFilterField();
            retentionFilterFunc();
            getVersions();
            calculateChartData(filteredRetention);
          }}
          role="button"
          on:keydown={(e) => {
            e.preventDefault();
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

  <button
    class="button"
    on:click={() => {
      showTable = true;
      showChart = false;
    }}>Table</button
  >
  <button
    class="button"
    on:click={() => {
      calculateChartData(filteredRetention);
      showTable = false;
      showChart = true;
    }}>Chart</button
  >

  {#if showTable}
    <Table {filteredRetention} />
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
    opacity: 1;
  }

  .button {
    padding: 1rem;
    border: none;
    background-color: rgb(2, 45, 100);
    color: #fff;
  }
</style>
