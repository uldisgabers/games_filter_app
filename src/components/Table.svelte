<script lang="ts">
  export let filteredRetention: Retention[];

  type Retention = {
    app_id: string;
    app_ver: string;
    country: string;
    days: number[];
  };
</script>

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

<style>
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
