<script>
  import { readNrbf } from './nrbf';
  import { JsonView } from '@zerodevx/svelte-json-view';

  let loading = false;
  let nrbfDataDebug = null;
  let nrbfData = null;
  let isTryingToDragOver = false;

  const loadNRBFData = file => {
    loading = true;
    const reader = new FileReader();
    reader.onload = loadEvent => {
      const raw = new Uint8Array(loadEvent.target.result);
      nrbfData = readNrbf(raw);
      console.log(nrbfData);
      nrbfDataDebug = raw.reduce((str, byte, idx) => {
        const char = String.fromCharCode(byte);
        const isReadable = char.match(/[ -~]/);
        const display = isReadable ? `${char} ` : `${byte.toString(16).padStart(2, '0')}`;
        let separator = idx % 20 ? ' ' : '\n';
        if (idx === nrbfData.offset) {
          separator = '#';
        }
        return `${str}${separator}${display}`;
      }, '');
      loading = false;
    }
    reader.readAsArrayBuffer(file);
  }

  const onNRBFDataLoad = $event => {
    const [file] = $event.target.files;
    if (!file) return;
    loadNRBFData(file);
  }

  const onFileDrop = $event => {
    $event.preventDefault();
    isTryingToDragOver = false;
    if (!$event.dataTransfer.items) return;
    loadNRBFData($event.dataTransfer.items[0].getAsFile());
  }

</script>

<main on:dragover={$event => {
  $event.preventDefault();
  isTryingToDragOver = true;
}} on:drop={onFileDrop}>
  <h1>📯 horny on main</h1>

  <label for="nrbfdata">
    A really hacky broken ass partially working NRBF viewer born from trying to pry Trombone Champ files open<br /><br />
    This was tested with:<br />
    <code>TromboneChamp\TromboneChamp_Data\StreamingAssets\leveldata\songdata.tchamp</code><br />
    <code>TromboneChamp\TromboneChamp_Data\StreamingAssets\leveldata\*.tmb</code><br />
    Recent Clone Hero PTB song caches (I don't know where they are)<br /><br />
  </label>
  {#if isTryingToDragOver}
    <div class="dropzone">Drop your file here!</div>
  {:else}
    <input id="nrbfdata" type="file" on:change={onNRBFDataLoad} accept="nrbfdata.tchamp" />
  {/if}

  {#if nrbfData}
    <div class="jsonview">
      <JsonView json={nrbfData} depth={1} />
    </div>

    <div>
      <details>
        <summary>Show raw data</summary>      
        <pre>{nrbfDataDebug}</pre>
      </details>
    </div>
  {:else if loading}
    <p>Loading, please wait...</p>
  {:else}
    <p>
      Data will be shown here<br />
      You can also open the browser console (F12) to have something better to play with
    </p>
  {/if}
</main>

<style>
  main {
    height: 100%;
  }

  .dropzone {
    margin: auto;
    width: 500px;
    height: 300px;
    border: 1px dashed azure;
    border-radius: 5px;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .jsonview {
    margin: 1em 0;
    text-align: left;
    font-family: 'Courier New', Courier, monospace;
    background: #eee;
    font-size: .8em;
  }
</style>
