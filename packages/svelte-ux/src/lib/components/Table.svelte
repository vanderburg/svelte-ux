<script lang="ts">
  import { createEventDispatcher } from 'svelte';

  import { tableCell } from '../actions/table.js';
  import type { ColumnDef } from '../types/table.js';
  import { cls } from '../utils/styles.js';

  import { getCellValue, getCellHeader, getHeaders, getRowColumns } from '../utils/table.js';

  import TableOrderIcon from './TableOrderIcon.svelte';
  import { getComponentClasses } from './theme.js';
  import { getSettings } from './settings.js';

  const dispatch = createEventDispatcher();

  export let columns: ColumnDef[] = [];
  export let data: any[] | null = [];
  export let orderBy: string | undefined = undefined;
  export let orderDirection: 'asc' | 'desc' | undefined = undefined;

  export let classes: {
    container?: string;
    wrapper?: string;
    table?: string;
    thead?: string;
    tbody?: string;
    tr?: string;
    th?: string;
    td?: string;
  } = {};
  // TODO: Figure out circular reference error
  // const settingsClasses = getComponentClasses('Table');
  const settingsClasses: typeof classes = {};

  export let styles: {
    container?: string;
    wrapper?: string;
    table?: string;
    thead?: string;
    tbody?: string;
    tr?: string;
    th?: string;
    td?: string;
  } = {};

  $: order = { by: orderBy, direction: orderDirection };

  $: headers = getHeaders(columns).map((headerRow) => {
    return headerRow.map((column) => {
      return {
        ...column,
        classes: {
          th: cls(settingsClasses.th, classes.th, column.classes?.th),
          td: cls(settingsClasses.td, classes.td, column.classes?.td),
        },
      };
    });
  });
  $: rowColumns = getRowColumns(columns).map((column) => {
    return {
      ...column,
      classes: {
        th: cls(settingsClasses.th, classes.th, column.classes?.th),
        td: cls(settingsClasses.td, classes.td, column.classes?.td),
      },
    };
  });

  const { format } = getSettings();
  $: getCellContent = (column: ColumnDef, rowData: any, rowIndex: number) => {
    let value = getCellValue(column, rowData, rowIndex);
    if (column.format) {
      if (typeof column.format === 'function') {
        return column.format(value, rowData, rowIndex);
      } else {
        return $format(value, column.format);
      }
    } else {
      return value;
    }
  };
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<div
  class={cls(
    'Table',
    'table-container',
    settingsClasses.container,
    classes.container,
    $$props.class
  )}
  style={styles.container}
>
  <div
    class={cls('table-wrapper', settingsClasses.wrapper, classes.wrapper)}
    style={styles.wrapper}
  >
    <table class={cls('w-full', settingsClasses.table, classes.table)} style={styles.table}>
      <slot name="headers" {headers} {getCellHeader}>
        <thead class={cls(settingsClasses.thead, classes.thead)} style={styles.thead}>
          {#each headers ?? [] as headerRow}
            <tr class={cls(settingsClasses.tr, classes.tr)} style={styles.tr}>
              {#each headerRow ?? [] as column}
                <th
                  use:tableCell={{ column }}
                  class="column-{column.name}"
                  class:whitespace-nowrap={orderBy}
                  style={styles.th}
                  on:click={(e) => dispatch('headerClick', { column })}
                >
                  {getCellHeader(column)}
                  <TableOrderIcon {order} {column} />
                </th>
              {/each}
            </tr>
          {/each}
        </thead>
      </slot>

      <slot />

      <slot name="data" {data} columns={rowColumns} {getCellValue} {getCellContent}>
        <tbody class={cls(settingsClasses.tbody, classes.tbody)} style={styles.tbody}>
          {#each data ?? [] as rowData, rowIndex}
            <tr class={cls(settingsClasses.tr, classes.tr)} style={styles.tr}>
              {#each rowColumns ?? [] as column}
                <td
                  use:tableCell={{ column, rowData, rowIndex, tableData: data }}
                  class="column-{column.name}"
                  style={styles.td}
                  on:click={(e) => dispatch('cellClick', { column, rowData })}
                >
                  {#if column.html}
                    {@html getCellContent(column, rowData, rowIndex)}
                  {:else}
                    {getCellContent(column, rowData, rowIndex)}
                  {/if}
                </td>
              {/each}
            </tr>
          {/each}
        </tbody>
      </slot>
    </table>
  </div>
</div>
