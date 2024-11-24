<script lang="ts">
  import { getContext, setContext } from 'svelte';

  // Props
  export let expanded = $state(false);
  export let key: string;

  // Context types
  type JsonViewerContext = {
    keyPath: string[];
    level: number;
    expanded: boolean;
    expandable: boolean;
  };

  // Get parent context
  const parentContext = getContext<JsonViewerContext>('jsonViewer');
  
  // Create new context state
  let expandable = $state(false);
  
  // Calculate new context values
  let newKeyPath = key !== '[[Entries]]' 
    ? [...parentContext.keyPath, key]
    : parentContext.keyPath;
    
  let newLevel = key !== '[[Entries]]'
    ? parentContext.level + 1
    : parentContext.level;

  // Set new context for children
  setContext<JsonViewerContext>('jsonViewer', {
    keyPath: newKeyPath,
    level: newLevel,
    expanded,
    expandable
  });
</script>

<slot />
