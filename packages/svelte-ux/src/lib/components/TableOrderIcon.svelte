<script lang="ts">
  import { mdiArrowUp } from '@mdi/js';

  import type { TableOrderState } from '../stores/tableOrderStore.js';
  import type { ColumnDef } from '../types/table.js';
  import Icon from '../components/Icon.svelte';

  export let order: TableOrderState;
  export let column: ColumnDef;
</script>

{#if order.by && (order.by === column.value || order.by === column.name || order.by === column.orderBy)}
  <div
    class="TableOrderIcon inline-block transition duration-100 transform"
    class:rotate-180={order.direction === 'desc'}
  >
    <Icon data={mdiArrowUp} />
  </div>
{/if}
