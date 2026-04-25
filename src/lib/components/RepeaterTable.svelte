<!-- RepeaterList.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import Papa from 'papaparse';

  interface Repeater {
    uid: number;
    type_of_station: string;
    band: string;
    frequency_tx: number;
    frequency_rx: number;
    ctcss_tx: number | null;
    ctcss_rx: number | null;
    ch: string;
    ch_new: string;
    callsign: string;
    antenna_heigth: number | null;
    site_name: string;
    sysop: string;
    url: string | null;
    hardware: string | null;
    mmdvm: boolean | null;
    solar_power: boolean | null;
    battery_power: boolean | null;
    status: string;
    fm: boolean | null;
    fm_wakeup: string | null;
    dmr: boolean | null;
    cc: number | null;
    c4fm: boolean | null;
    c4fm_groups: string | null;
    dstar: boolean | null;
    dstar_rpt1: string | null;
    dstar_rpt2: string | null;
    tetra: boolean | null;
    other_mode: boolean | null;
    other_mode_name: string | null;
    echolink: string | null;
    echolink_id: number | null;
    digital_id: string | null;
    comment: string | null;
  }

  // State declarations
  let repeaters = $state<Repeater[]>([]);
  let filteredRepeaters = $state<Repeater[]>([]);
  let searchQuery = $state('');
  let selectedType = $state('Alle Typen');
  let selectedModes = $state<string[]>(['Alle']);
  let selectedBands = $state<string[]>(['Alle']);
  let sortField = $state('callsign');
  let sortDirection = $state(1);
  let stationTypes = $state<string[]>(['all']);
  let availableModes = $state<string[]>([]);
  let availableBands = $state<string[]>([]);

  const fieldsToFilter = [
    'callsign', 
    'site_name',
    'frequency_tx',
    'frequency_rx',
    'ctcss_tx',
    'ctcss_rx'
  ]

  // German translations for station types
  const typeTranslations = {
    'all': 'Alle Typen',
    'repeater_voice': 'Sprechfunk',
    'beacon': 'Baken',
    'digipeater': 'Digipeater',
    'atv': 'ATV'
  };

  let selectedStatus = $state('active');

  // Status options in German
  const statusOptions = {
    'all': 'Alle',
    'active': 'Aktiv',
    'inactive': 'Inaktiv',
    'testing': 'Test',
    'planned': 'Geplant',
    'historic': 'Historisch'
  };


  // Get unique station types
  $effect(() => {
    stationTypes = ['Alle Typen', ...new Set(repeaters.map(r => r.type_of_station))];
  });

  // Get unique bands
  $effect(() => {
   availableBands = ['Alle', ...new Set(repeaters.map(r => r.band))];
  });

  // Get all possible modes
  $effect(() => {
    availableModes = [
      'Alle',  
      ...new Set([
        ...(repeaters.filter(r => r.fm).map(() => 'FM')),
        ...(repeaters.filter(r => r.dmr).map(() => 'DMR')),
        ...(repeaters.filter(r => r.c4fm).map(() => 'C4FM')),
        ...(repeaters.filter(r => r.dstar).map(() => 'D-STAR')),
        ...(repeaters.filter(r => r.tetra).map(() => 'TETRA')),
        ...(repeaters.filter(r => r.echolink_id).map(() => 'ECHOLINK')),
        ...(repeaters.filter(r => r.other_mode && r.other_mode_name)
            .map(r => r.other_mode_name))
      ])
    ];
  });

  // Filter and sort effect
  $effect(() => {
    filteredRepeaters = repeaters
      .filter(repeater => {
        const matchesSearch = 
        Object.entries(repeater)
        .filter(([k, v]) => fieldsToFilter.includes(k))
          .join(' ')
          .toLowerCase()
          .includes(searchQuery.toLowerCase());

        const matchesType = selectedType === 'Alle Typen' || 
          repeater.type_of_station === selectedType;

        const matchesStatus = selectedStatus === 'all' || 
          repeater.status === selectedStatus;

        const matchesModes = selectedModes.length === 0 || 
          selectedModes.some(mode => {
            switch(mode) {
              case 'Alle': return repeater;
              case 'FM': return repeater.fm;
              case 'DMR': return repeater.dmr;
              case 'C4FM': return repeater.c4fm;
              case 'D-STAR': return repeater.dstar;
              case 'TETRA': return repeater.tetra;
              case 'ECHOLINK': return repeater.echolink_id;
              default: return repeater.other_mode_name === mode;
            }
          });

          const matchesBands = selectedBands.length === 0 || 
          selectedBands.some(band => { 
            switch(band) {
              case 'Alle': return repeater;
              default: return repeater.band === band; 
            }

          });

        return matchesSearch && matchesType && matchesStatus && matchesModes && matchesBands;
      })
      .sort((a, b) => {
        return (a[sortField] > b[sortField] ? 1 : -1) * sortDirection;
      });
  });
  
  onMount(async () => {
    try {
      const response = await fetch('https://repeater.oevsv.at/api/trx');
      repeaters = await response.json();
    } catch (error) {
      console.error('Error fetching repeater data:', error);
    }
  });

  function exportToCsv() {
    const csv = Papa.unparse(filteredRepeaters);
    const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
    const link = document.createElement('a');
    link.href = URL.createObjectURL(blob);
    link.download = 'repeaters.csv';
    link.click();
  }

  function handleSort(field: string) {
    if (sortField === field) {
      sortDirection *= -1;
    } else {
      sortField = field;
      sortDirection = 1;
    }
  }

  function getTooltipInfo(r: Repeater): { label: string; value: string }[] {
    const items: { label: string; value: string }[] = [];
    if (r.sysop) items.push({ label: 'Sysop', value: r.sysop });
    if (r.hardware) items.push({ label: 'Hardware', value: r.hardware });
    if (r.mmdvm != null) items.push({ label: 'MMDVM', value: r.mmdvm ? 'Ja' : 'Nein' });
    if (r.solar_power) items.push({ label: 'Solar', value: 'Ja' });
    if (r.battery_power) items.push({ label: 'Akku', value: 'Ja' });
    if (r.antenna_heigth != null) items.push({ label: 'Antennenh\u00f6he', value: `${r.antenna_heigth} m` });
    if (r.fm_wakeup) items.push({ label: 'FM Wakeup', value: r.fm_wakeup });
    if (r.digital_id) items.push({ label: 'Digital ID', value: r.digital_id });
    if (r.cc != null) items.push({ label: 'CC', value: String(r.cc) });
    if (r.c4fm_groups) items.push({ label: 'C4FM Groups', value: r.c4fm_groups });
    if (r.dstar_rpt1) items.push({ label: 'D-STAR RPT1', value: r.dstar_rpt1 });
    if (r.dstar_rpt2) items.push({ label: 'D-STAR RPT2', value: r.dstar_rpt2 });
    if (r.echolink) items.push({ label: 'Echolink', value: r.echolink });
    if (r.comment) items.push({ label: 'Kommentar', value: r.comment });
    return items;
  }
</script>
<div class="container">
  <div class="header">
    <img 
      src="raute.png" 
      alt="ÖVSV Logo" 
      class="logo"
    />
    <div>
      <h2>Repeater Liste</h2>
      <p class="info">Entwicklung: <a href="https://github.com/OE3ANC/oevsv-repeaterliste">OE3ANC</a></p>
      <p class="info">Daten: <a href="https://www.oevsv.at/funkbetrieb/ukw-referat/maps/">ÖVSV DV</a> (CC BY 4.0)</p>
    </div>
  </div>
  
  <div class="filters">
    <input
      type="text"
      bind:value={searchQuery}
      placeholder="Suchen..."
      class="search-input"
    />

    <select bind:value={selectedType} class="select">
      {#each stationTypes as type}
        <option value={type}>{typeTranslations[type] || type}</option>
      {/each}
    </select>

    <select
      multiple
      bind:value={selectedBands}
      class="select"
    >
      {#each availableBands as band}
        <option value={band}>{band}</option>
      {/each}
    </select>

    <select
      multiple
      bind:value={selectedModes}
      class="select"
    >
      {#each availableModes as mode}
        <option value={mode}>{mode}</option>
      {/each}
    </select>

    <select bind:value={selectedStatus} class="select">
      {#each Object.entries(statusOptions) as [value, label]}
        <option value={value}>{label}</option>
      {/each}
    </select>

    <button onclick={exportToCsv} class="export-btn">
      CSV Export
    </button>
  </div>

  <div class="table-container">
    <table>
      <thead>
        <tr>
          <th onclick={() => handleSort('callsign')}>Rufzeichen</th>
          <th onclick={() => handleSort('type_of_station')}>Typ</th>
          <th onclick={() => handleSort('band')}>Band</th>
          <th onclick={() => handleSort('frequency_tx')}>TX (MHz)</th>
          <th onclick={() => handleSort('frequency_rx')}>RX (MHz)</th>
          <th onclick={() => handleSort('ctcss_tx')}>CTCSS TX</th>
          <th onclick={() => handleSort('ctcss_rx')}>CTCSS RX</th>
          <th onclick={() => handleSort('site_name')}>Standort</th>
          <th>Modi</th>
          <th>Echolink</th>
        </tr>
      </thead>
      <tbody>
        {#each filteredRepeaters as repeater}
          {@const tooltipInfo = getTooltipInfo(repeater)}
          <tr>
            <td class="callsign-cell">
              {#if repeater.url}
                <a href={repeater.url} target="_blank" rel="noopener" class="callsign-link">{repeater.callsign}</a>
              {:else}
                {repeater.callsign}
              {/if}
              {#if tooltipInfo.length > 0}
                <div class="tooltip">
                  {#each tooltipInfo as item}
                    <div class="tooltip-row">
                      <span class="tooltip-label">{item.label}:</span>
                      {item.value}
                    </div>
                  {/each}
                </div>
              {/if}
            </td>
            <td>{typeTranslations[repeater.type_of_station] || repeater.type_of_station}</td>
            <td>{repeater.band}</td>
            <td>{repeater.frequency_tx}</td>
            <td>{repeater.frequency_rx}</td>
            <td>{repeater.ctcss_tx}</td>
            <td>{repeater.ctcss_rx}</td>
            <td>{repeater.site_name}</td>
            <td>
              {[
                repeater.fm && 'FM',
                repeater.dmr && 'DMR',
                repeater.c4fm && 'C4FM',
                repeater.dstar && 'D-STAR',
                repeater.tetra && 'TETRA',
                repeater.other_mode && repeater.other_mode_name
              ].filter(Boolean).join(', ')}
            </td>
            <td>{repeater.echolink_id}</td>
          </tr>
        {/each}
      </tbody>
    </table>
  </div>
</div>
<style>

  .container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 1rem;
    font-family: 'Open Sans', sans-serif;
    transition: color 0.3s;
  }

  .header {
    display: flex;
    align-items: center;
    gap: 1rem;
    margin-bottom: 2rem;
  }

  .logo {
    height: 60px;
    width: auto;
  }

  h2 {
    margin: 0;
    color: var(--color-text);
  }

  .filters {
    display: flex;
    flex-wrap: wrap;
    gap: 1rem;
    margin-bottom: 1rem;
  }

  .select,
  .search-input {
    padding: 0.5rem;
    border: 1px solid var(--color-border);
    border-radius: 4px;
    min-width: 150px;
    flex: 1;
    background-color: var(--color-input-bg);
    color: var(--color-input-text);
    transition: border-color 0.3s, background-color 0.3s, color 0.3s;
  }

  .info {
    font-size: xx-small;
  }

  a {
    color: var(--color-link);
  }

  .export-btn {
    background-color: var(--color-button-bg);
    color: var(--color-button-text);
    border: none;
    padding: 0.5rem 1rem;
    border-radius: 4px;
    cursor: pointer;
    white-space: nowrap;
    transition: background-color 0.3s;
  }

  .export-btn:hover {
    background-color: var(--color-button-hover);
  }

  .table-container {
    width: 100%;
    overflow-x: auto;
    -webkit-overflow-scrolling: touch;
  }

  table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 1rem;
    min-width: 800px;
  }

  th, td {
    padding: 0.75rem;
    text-align: left;
    border-bottom: 1px solid var(--color-border);
    transition: border-color 0.3s;
  }

  th {
    background-color: var(--color-table-header-bg);
    cursor: pointer;
    white-space: nowrap;
    transition: background-color 0.3s;
  }

  th:hover {
    background-color: var(--color-table-header-hover);
  }

  tr:hover {
    background-color: var(--color-table-row-hover);
  }

  tr {
    transition: background-color 0.3s;
  }

  .callsign-cell {
    position: relative;
    cursor: help;
  }

  .tooltip {
    display: none;
    position: absolute;
    bottom: 100%;
    left: 0;
    z-index: 1000;
    background: var(--color-bg);
    color: var(--color-text);
    border: 1px solid var(--color-border);
    border-radius: 6px;
    padding: 0.75rem;
    white-space: nowrap;
    box-shadow: 0 4px 12px rgba(0,0,0,0.15);
    font-size: 0.8rem;
    pointer-events: none;
  }

  .callsign-cell:hover .tooltip {
    display: block;
  }

  .tooltip-row {
    margin-bottom: 0.25rem;
  }

  .tooltip-row:last-child {
    margin-bottom: 0;
  }

  .tooltip-label {
    font-weight: 600;
  }

  @media (max-width: 768px) {
    .header {
      flex-direction: column;
      text-align: center;
    }

    .logo {
      height: 50px;
    }

    .filters {
      flex-direction: column;
    }

    .search-input,
    .select,
    .export-btn {
      width: 100%;
    }

    th, td {
      padding: 0.5rem;
    }
  }
</style>