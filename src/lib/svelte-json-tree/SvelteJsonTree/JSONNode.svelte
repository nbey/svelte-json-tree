<script lang="ts">
  import JSONObjectNode from './JSONObjectNode.svelte';
  import JSONArrayNode from './JSONArrayNode.svelte';
  import JSONIterableArrayNode from './JSONIterableArrayNode.svelte';
  import JSONIterableMapNode from './JSONIterableMapNode.svelte';
  import JSONValueNode from './JSONValueNode.svelte';
  import ErrorNode from './ErrorNode.svelte';
  import objType from './utils/objType';
  import JsonStringNode from './JSONStringNode.svelte';
  import JsonFunctionNode from './JSONFunctionNode.svelte';
  import JsonSvelteStoreNode from './JSONSvelteStoreNode.svelte';
  import TypedArrayNode from './TypedArrayNode.svelte';
  import RegExpNode from './RegExpNode.svelte';
  import { getContext } from 'svelte';

  // Props
  export let value: unknown;

  // State using new runes syntax
  let nodeType = $state('');
  const { shouldTreatIterableAsObject } = getContext<{ shouldTreatIterableAsObject: boolean }>('jsonViewer');

  // Derived values using new $ syntax
  $: nodeType = objType(value, shouldTreatIterableAsObject);
  $: [componentType, props] = getComponentAndProps(nodeType, value);

  function getComponentAndProps(nodeType: string, value: any): [any, Record<string, any>?] {
    switch (nodeType) {
      case 'Object':
        if (typeof value.subscribe === 'function') return [JsonSvelteStoreNode, {}];
        return [JSONObjectNode, {}];
      case 'Error':
        return [ErrorNode, {}];
      case 'Array':
        return [JSONArrayNode, {}];
      case 'Map':
        return [JSONIterableMapNode, {}];
      case 'Iterable':
      case 'Set':
        return [JSONIterableArrayNode, { nodeType }];
      case 'Number':
        return [JSONValueNode, { nodeType }];
      case 'String':
        return [JsonStringNode, {}];
      case 'Boolean':
        return [JSONValueNode, { nodeType, value: value ? 'true' : 'false' }];
      case 'Date':
        return [JSONValueNode, { nodeType, value: value.toISOString() }];
      case 'Null':
        return [JSONValueNode, { nodeType, value: 'null' }];
      case 'Undefined':
        return [JSONValueNode, { nodeType, value: 'undefined' }];
      case 'Function':
      case 'AsyncFunction':
      case 'AsyncGeneratorFunction':
      case 'GeneratorFunction':
        return [JsonFunctionNode, {}];
      case 'Symbol':
        return [JSONValueNode, { nodeType, value: value.toString() }];
      case 'BigInt':
        return [JSONValueNode, { nodeType, value: String(value) + 'n' }];
      case 'ArrayBuffer':
        return [JSONValueNode, { nodeType, value: `ArrayBuffer(${value.byteLength})` }];
      case 'BigInt64Array':
      case 'BigUint64Array':
      case 'Float32Array':
      case 'Float64Array':
      case 'Int8Array':
      case 'Int16Array':
      case 'Int32Array':
      case 'Uint8Array':
      case 'Uint8ClampedArray':
      case 'Uint16Array':
      case 'Uint32Array':
        return [TypedArrayNode, { nodeType }];
      case 'RegExp':
        return [RegExpNode, {}];
      default:
        return [JSONObjectNode, { summary: nodeType }];
    }
  }
</script>

<svelte:component this={componentType} {value} {...props} />
