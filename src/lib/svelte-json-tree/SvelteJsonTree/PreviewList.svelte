<script lang="ts">
  import { getContext } from 'svelte';

  // Props with TypeScript types
  export let list:
    | Array<unknown>
    | Int8Array
    | Uint8Array
    | Uint8ClampedArray
    | Int16Array
    | Uint16Array
    | Int32Array
    | Uint32Array
    | Float32Array
    | Float64Array
    | BigInt64Array
    | BigUint64Array;
  export let hasMore: boolean;
  export let label: string | undefined = undefined;
  export let prefix: string | undefined = undefined;
  export let postfix: string | undefined = undefined;
  export let root: boolean = false;
  
  // Get context
  type JsonViewerContext = {
    showPreview: boolean;
  };
  
  const { showPreview } = getContext<JsonViewerContext>('jsonViewer');
</script>

{#if root || showPreview}
  {#if prefix}
    {#if label}
      <span class="label">{label}</span>
    {/if}
    <span class="operator">{prefix}</span>
  {/if}
  
  {#each list as item, index}
    <slot name="item" {item} {index} />
    {#if index < list.length - 1}
      <span class="comma operator">,</span>
    {/if}
  {/each}
  
  {#if hasMore}
    <span class="comma operator">,</span>
    <span class="operator">…</span>
  {/if}
  
  {#if postfix}
    <span class="operator">{postfix}</span>
  {/if}
{/if}

<style>
  .comma {
    margin-left: -0.5em;
    margin-right: 0.5em;
  }
  
  .operator {
    color: var(--operator-color, inherit);
  }
  
  .label {
    color: var(--label-color, inherit);
  }
</style>
