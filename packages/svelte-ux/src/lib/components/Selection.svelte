<script lang="ts">
  import selectionStore from '../stores/selectionStore.js';

  type T = $$Generic;

  export let initial: any[] = [];
  export let all: any[] = [];
  export let single = false;
  export let max: number | undefined = undefined;

  const selection = selectionStore({ initial, all, single, max });
  $: $selection.all.set(all);
</script>

<!-- TODO: `<slot {...$selection} />` does not play well with sveld -->
<slot
  selected={$selection.selected}
  isSelected={$selection.isSelected}
  isDisabled={$selection.isDisabled}
  toggleAll={$selection.toggleAll}
  toggleSelected={$selection.toggleSelected}
  isAllSelected={$selection.isAllSelected}
  isAnySelected={$selection.isAnySelected}
  clear={$selection.clear}
/>
