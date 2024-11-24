<script lang="ts">
  import { onMount } from 'svelte';
  import { getContext, setContext } from 'svelte';
  import JSONArrow from './JSONArrow.svelte';
  import Summary from './Summary.svelte';
  import Expandable from './Expandable.svelte';

  // Props
  export let keys: string[];
  export let shouldShowColon: (key: string) => boolean = undefined;
  export let expandKey = (key: string) => key;
  export let defaultExpanded = false;

  // Context types
  type JsonViewerContext = {
    isParentExpanded: boolean;
    displayMode: 'summary' | 'full';
    root: boolean;
    expanded: boolean;
    expandable: boolean;
    keyPath: string[];
    level: number;
    shouldExpandNode: (params: { keyPath: string[]; level: number }) => boolean | undefined;
  };

  // State
  let expanded = $state(false);
  let expandable = $state(true);
  let childExpanded = $state<boolean[]>([]);

  // Get parent context
  const context = getContext<JsonViewerContext>('jsonViewer');
  const {
    isParentExpanded,
    displayMode,
    root = false,
    keyPath,
    level,
    shouldExpandNode
  } = context;

  // Initialize child expanded states
  $: childExpanded = keys.map(() => false);

  // Set expandable state
  $: expandable = true;

  // Handle initial expansion state
  $: if (displayMode !== 'summary') {
    if (!defaultExpanded) {
      const shouldExpand = shouldExpandNode({ keyPath, level });
      if (shouldExpand !== undefined) {
        defaultExpanded = shouldExpand;
      }
    }
  }

  // Setup parent expansion listener
  onMount(() => {
    if (displayMode !== 'summary') {
      // Watch parent expanded state
      const unsubscribe = watch(() => {
        if (!isParentExpanded) {
          expanded = false;
        } else {
          expanded = defaultExpanded;
        }
      });
      
      return unsubscribe;
    }
  });

  // Event handlers
  function toggleExpand(event?: Event) {
    if (event) {
      event.stopPropagation();
    }
    expanded = !expanded;
  }

  function toggleChildExpand(index: number) {
    childExpanded[index] = !childExpanded[index];
  }

  // Set context for children
  $: setContext<JsonViewerContext>('jsonViewer', {
    ...context,
    expanded,
    expandable,
    isParentExpanded: expanded
  });
</script>

{#if displayMode === 'summary'}
  <slot name="summary" />
{:else}
  <!-- svelte-ignore a11y-click-events-have-key-events a11y-no-static-element-interactions -->
  <span class="root" on:click={toggleExpand}>
    {#if root}
      <JSONArrow expanded={expanded} />
    {/if}
    <Summary>
      <slot name="preview" {root} />
    </Summary>
  </span>

  {#if expanded}
    <!-- svelte-ignore a11y-click-events-have-key-events a11y-no-noninteractive-element-interactions -->
    <ul on:click|stopPropagation={toggleExpand}>
      {#each keys as key, index}
        <!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
        <li class:indent={expanded} on:click|stopPropagation={() => {}}>
          <Expandable key={expandKey(key)} expanded={childExpanded[index]}>
            <!-- svelte-ignore a11y-no-static-element-interactions -->
            <span 
              class="label" 
              on:click={() => toggleChildExpand(index)}
            >
              <JSONArrow />
              <slot name="item_key" {key} {index} />
              {#if !shouldShowColon || shouldShowColon(key)}
                <span class="operator">{': '}</span>
              {/if}
            </span>
            <slot name="item_value" {key} {index} />
          </Expandable>
        </li>
      {/each}
    </ul>
  {/if}
{/if}

<style>
  .root {
    display: inline-block;
    position: relative;
  }

  .indent {
    padding-left: var(--li-identation);
  }

  .label {
    position: relative;
  }

  .operator {
    color: var(--operator-color, inherit);
  }
</style>
