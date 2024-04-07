<script>
  
</script>

<div class="dropdown">
  <input
    placeholder={selectedGame}
    class="dropbtn"
    on:input={handleGameInput}
  />
  <div class="dropdown-content">
    <a
      href=" "
      on:click={(e) => {
        e.preventDefault();
        filteredRetention = retention;
        selectedGame = "All";
        isGameSelected = false;
        console.log(filteredRetention);
      }}
    >
      All
    </a>
    {#each filteredGames as game}
      <a
        href=" "
        on:click={(e) => {
          e.preventDefault();
          selectedGame = game.name;
          isGameSelected = true;

          filteredRetention = retention.filter((item) => {
            return game.app_id === item.app_id;
          });
          console.log(filteredRetention);
        }}
      >
        <img src={game.icon} width="20" height="20" alt={game.name} />
        {game.name}</a
      >
    {/each}
  </div>
</div>

<div class="dropdown">
  <input
    placeholder={selectedVersion}
    class="dropbtn"
    on:input={handleVersionInput}
  />
  <div class="dropdown-content">
    <a
      href=" "
      on:click={(e) => {
        e.preventDefault();
        selectedVersion = "All";
        // filteredRetention = retention;
        // selectedGame = "All";
        // isGameSelected = false;
        // console.log(filteredRetention);
      }}
    >
      All
    </a>
    {#each getVersions() as version}
      <a
        href=" "
        on:click={(e) => {
          e.preventDefault();
          selectedVersion = version;
        }}
      >
        <b>{version}</b> ({getDevicesForVersionsCount(
          filteredRetention,
          version,
        )})</a
      >
    {/each}
  </div>
</div>

<select disabled={!isGameSelected} name="version" id="version">
  <option value="all">All</option>
  <input type="text" />
  {#each getVersions() as version}
    <option value={version}
      >{version} ({getDevicesForVersionsCount(
        filteredRetention,
        version,
      )})</option
    >
  {/each}
</select>

<select name="country" id="country">
  <option value="all">All</option>
  {#each countries as country}
    <option value={country}>{country}</option>
  {/each}
</select>

<br />