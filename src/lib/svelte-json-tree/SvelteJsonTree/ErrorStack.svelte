<script lang="ts">
  import JsonNode from './JSONNode.svelte';
  import { getContext } from 'svelte';

  // Props
  export let stack: string[];

  // Get context
  type JsonViewerContext = {
    expanded: boolean;
    expandable: boolean;
  };
  
  const context = getContext<JsonViewerContext>('jsonViewer');
  let { expanded, expandable } = context;

  // Set expandable state
  $: expandable = true;

  // Click handler
  function toggleExpanded() {
    expanded = !expanded;
  }
</script>

<!-- svelte-ignore a11y-click-events-have-key-events a11y-no-static-element-interactions -->
<span on:click={toggleExpanded}>
  {#if expanded}
    {#each stack as line, index}
      {@const appendNewLine = index < stack.length - 1}
      <span 
        class:indent={index > 0}
      >
        <JsonNode 
          value={line + (appendNewLine ? '\\n' : '')} 
        />
        <span class="operator">
          {appendNewLine ? ' +' : ''}
        </span>
      </span>
      <br />
    {/each}
  {:else}
    <span>
      <JsonNode value={stack[0] + '…'} />
    </span>
  {/if}
</span>

<style>
  .indent {
    padding-left: var(--li-identation);
  }
  
  .operator {
    color: var(--operator-color, inherit);
  }
</style>
