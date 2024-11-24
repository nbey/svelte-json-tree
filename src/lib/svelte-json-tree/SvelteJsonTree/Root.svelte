<script lang="ts">
  import JSONNode from './JSONNode.svelte';
  import Expandable from './Expandable.svelte';
  import { getShouldExpandNode } from './utils/expand';

  // Props
  export let value: unknown;
  export let shouldShowPreview: boolean = true;
  export let shouldTreatIterableAsObject: boolean = false;
  export let defaultExpandedPaths: string[] = [];
  export let defaultExpandedLevel: number = 0;

  // State
  let expanded = $state(true);
  
  // Derived values
  $derived(expandable = value && typeof value === 'object');
  $derived(shouldExpandNode = getShouldExpandNode({ defaultExpandedPaths, defaultExpandedLevel }));

  // Context values - using Svelte 5's new context system
  const context = $state({
    expanded,
    isParentExpanded: true,  // Previously a readable store, now just a boolean
    root: true,
    shouldExpandNode: (opts) => shouldExpandNode(opts),
    level: 0,
    keyPath: [],
    showPreview: shouldShowPreview,
    shouldTreatIterableAsObject,
  });

  // Replace useState with setContext
  $effect(() => {
    setContext('json-tree', context);
  });
</script>

<div class:expandable>
  {#if expandable}
    <Expandable key="$" expanded={$expanded}>
      <JSONNode {value} />
    </Expandable>
  {:else if typeof value === 'string'}
    <span>{value}</span>
  {:else}
    <JSONNode {value} />
  {/if}
</div>

<style>
  div {
    --string-color: var(--json-tree-string-color, #cb3f41);
    --symbol-color: var(--json-tree-symbol-color, #cb3f41);
    --boolean-color: var(--json-tree-boolean-color, #112aa7);
    --function-color: var(--json-tree-function-color, #112aa7);
    --number-color: var(--json-tree-number-color, #3029cf);
    --label-color: var(--json-tree-label-color, #871d8f);
    --property-color: var(--json-tree-property-color, #000000);
    --arrow-color: var(--json-tree-arrow-color, #727272);
    --operator-color: var(--json-tree-operator-color, #727272);
    --null-color: var(--json-tree-null-color, #8d8d8d);
    --undefined-color: var(--json-tree-undefined-color, #8d8d8d);
    --date-color: var(--json-tree-date-color, #8d8d8d);
    --internal-color: var(--json-tree-internal-color, grey);
    --regex-color: var(--json-tree-regex-color, var(--string-color));
    --li-identation: var(--json-tree-li-indentation, 1em);
    --li-line-height: var(--json-tree-li-line-height, 1.3);
    font-size: var(--json-tree-font-size, 12px);
    font-family: var(--json-tree-font-family, 'Courier New', Courier, monospace);
  }
  div :global(li) {
    line-height: var(--li-line-height);
    display: var(--li-display, list-item);
    list-style: none;
  }
  div,
  div :global(ul) {
    padding: 0;
    margin: 0;
  }
  .expandable {
    margin-left: var(--li-identation);
  }
  div {
    cursor: default;
  }
  div :global(.label) {
    color: var(--label-color);
  }
  div :global(.property) {
    color: var(--property-color);
  }
  div :global(.internal) {
    color: var(--internal-color);
  }
  div :global(.operator) {
    color: var(--operator-color);
  }
  span {
    white-space: pre-wrap;
  }
</style>
