<script lang="ts">
  import { getContext } from 'svelte';

  // Props
  export let value: string;

  // Context type
  type JsonViewerContext = {
    displayMode: 'summary' | 'full';
  };

  // Character map for escaping
  const map: Record<string, string> = {
    '\n': '\\n',
    '\t': '\\t',
    '\r': '\\r',
  };

  // Get display mode from context
  const { displayMode } = getContext<JsonViewerContext>('jsonViewer');

  // Compute serialized value
  $: serialised = value.replace(/[\n\t\r]/g, (char) => map[char]);

  // Compute displayed string
  $: displayedString = displayMode === 'summary' && serialised.length > 30
    ? serialised.slice(0, 30) + '…'
    : serialised;
</script>

<span>"{displayedString}"</span>

<style>
  span {
    color: var(--string-color, #22863a);
    word-break: break-all;
    word-wrap: break-word;
  }

  :global([data-theme="dark"]) span {
    color: var(--string-color, #7ec699);
  }
</style>
