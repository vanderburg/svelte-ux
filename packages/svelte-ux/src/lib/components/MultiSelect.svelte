<script lang="ts">
  import { getComponentClasses } from './theme.js';

  import { type ComponentProps, createEventDispatcher } from 'svelte';
  import { flip } from 'svelte/animate';
  import { get, partition } from 'lodash-es';

  import { mdiMagnify } from '@mdi/js';

  import Button from './Button.svelte';
  import InfiniteScroll from './InfiniteScroll.svelte';
  import MultiSelectOption from './MultiSelectOption.svelte';
  import TextField from './TextField.svelte';

  import dirtyStore from '../stores/dirtyStore.js';
  import selectionStore from '../stores/selectionStore.js';
  import uniqueStore from '../stores/uniqueStore.js';
  import { cls } from '$lib/utils/styles.js';

  type Option = $$Generic;

  export let options: Option[];
  export let value: string[] | number[] = [];
  export let indeterminateSelected: string[] | number[] = [];
  export let duration = 200;
  export let inlineSearch = false;
  export let autoFocusSearch = false;
  export let placeholder = 'Search items';

  /** Wrap options in `InfiniteScroll` to amortize rendering of a large number of options */
  export let infiniteScroll = false;

  /** Maximum number of options that can be selected  */
  export let max: number | undefined = undefined;

  export let labelProp = 'name';
  export let valueProp = 'value';

  export let cancelButtonProps: ComponentProps<Button> | undefined = undefined;
  export let applyButtonProps: ComponentProps<Button> | undefined = undefined;

  export let classes: {
    root?: string;
  } = {};
  const settingsClasses = getComponentClasses('MultiSelect');

  export let onApply = async (ctx: {
    selection: typeof $selection;
    indeterminate: typeof $indeterminateStore;
    original: { selected: Option[]; unselected: Option[] };
  }) => {
    // no-op by default
  };

  const dispatch = createEventDispatcher<{
    change: {
      selection: typeof $selection;
      indeterminate: typeof $indeterminateStore;
      original: { selected: Option[]; unselected: Option[] };
    };
    cancel: null;
  }>();

  export let searchText = '';
  let applying = false;

  // Partition options based on if they initially selected, which will be displayed at the top
  $: [selectedOptions, unselectedOptions] = partition(options, (x) =>
    value.includes(get(x, valueProp))
  );

  // Filter by search text
  function applyFilter(option: Option, searchText: string) {
    if (searchText) {
      return get(option, labelProp).toLowerCase().includes(searchText.toLowerCase());
    } else {
      // show all if no search set
      return true;
    }
  }
  $: filteredSelectedOptions = selectedOptions.filter((x) => applyFilter(x, searchText));
  $: filteredUnselectedOptions = unselectedOptions.filter((x) => applyFilter(x, searchText));

  $: selection = selectionStore({
    initial: selectedOptions.map((x) => get(x, valueProp)),
    max,
  });

  $: isSelectionDirty = dirtyStore(selection);
  $: indeterminateStore = uniqueStore(indeterminateSelected);

  function onChange() {
    const changeContext = {
      value: $selection.selected,
      selection: $selection,
      indeterminate: $indeterminateStore,
      original: { selected: selectedOptions, unselected: unselectedOptions },
    };
    dispatch('change', changeContext);
    searchText = '';
    // Store will be recreated when `selectedOptions` is updated, but just in case
    // `setTimeout` is used to delay the disabling the Apply button 1 event loop, otherwise if `type="submit" will not react the `<form on:submit>` - https://svelte.dev/repl/0875574693e14fa9a4348075b1788d0a?version=3.58.0
    setTimeout(() => {
      isSelectionDirty.reset();
    });
  }

  export function clear() {
    $selection.clear();
    onChange();
  }
</script>

{#if inlineSearch}
  <div class="border-b border-surface-content/10 p-4 pb-2">
    <TextField
      {placeholder}
      iconRight={mdiMagnify}
      bind:value={searchText}
      autofocus={{ delay: 100, disabled: !autoFocusSearch }}
    />
  </div>
{/if}

<div class={cls('overflow-auto py-1 px-4', settingsClasses.root, classes.root, $$restProps.class)}>
  <slot name="beforeOptions" selection={$selection} />

  <!-- initially selected options -->
  <InfiniteScroll items={filteredSelectedOptions} disabled={!infiniteScroll} let:visibleItems>
    {#each visibleItems as option (get(option, valueProp))}
      {@const label = get(option, labelProp)}
      {@const value = get(option, valueProp)}
      {@const checked = $selection.isSelected(value)}
      {@const indeterminate = $indeterminateStore.has(value)}
      {@const disabled = $selection.isDisabled(value)}
      {@const onChange = () => {
        // TODO: Try to figure out how to keep underling Checkbox controlled so state goes `indeterminate` => `checked` => `unchecked`
        // If partial/indeterminate, transition to fully selected, then deselect/select as usual
        // if ($indeterminateStore.has(value)) {
        //   indeterminateStore.delete(value);
        // } else {
        //   $selection.toggleSelected(value);
        // }

        // Clear indeterminate status and toggle `unchecked` (and will proceed to toggle `checked` => `unchecked` => etc)
        indeterminateStore.delete(value);
        $selection.toggleSelected(value);
      }}
      <div animate:flip={{ duration }}>
        <slot
          name="option"
          {option}
          {label}
          {value}
          {checked}
          {indeterminate}
          {disabled}
          {onChange}
        >
          <MultiSelectOption {checked} {indeterminate} {disabled} on:change={onChange}>
            {label}
          </MultiSelectOption>
        </slot>
      </div>
    {/each}
  </InfiniteScroll>

  {#if filteredSelectedOptions.length && filteredUnselectedOptions.length}
    <!-- separator between selected and deselected -->
    <div class="border-b border-surface-content/10" />
  {/if}

  <!-- initially unselected options -->
  <InfiniteScroll items={filteredUnselectedOptions} disabled={!infiniteScroll} let:visibleItems>
    {#each visibleItems as option (get(option, valueProp))}
      {@const label = get(option, labelProp)}
      {@const value = get(option, valueProp)}
      {@const checked = $selection.isSelected(value)}
      {@const indeterminate = $indeterminateStore.has(value)}
      {@const disabled = $selection.isDisabled(value)}
      {@const onChange = () => {
        // TODO: Try to figure out how to keep underling Checkbox controlled so state goes `indeterminate` => `checked` => `unchecked`
        // If partial/indeterminate, transition to fully selected, then deselect/select as usual
        // if ($indeterminateStore.has(value)) {
        //   indeterminateStore.delete(value);
        // } else {
        //   $selection.toggleSelected(value);
        // }

        // Clear indeterminate status and toggle `unchecked` (and will proceed to toggle `checked` => `unchecked` => etc)
        indeterminateStore.delete(value);
        $selection.toggleSelected(value);
      }}
      <div animate:flip={{ duration }}>
        <slot
          name="option"
          {option}
          {label}
          {value}
          {checked}
          {indeterminate}
          {disabled}
          {onChange}
        >
          <MultiSelectOption {checked} {indeterminate} {disabled} on:change={onChange}>
            {label}
          </MultiSelectOption>
        </slot>
      </div>
    {:else}
      {#if !filteredSelectedOptions.length}
        <div class="text-surface-content/50 text-xs py-2">There are no matching items.</div>
      {/if}
    {/each}
  </InfiniteScroll>

  <slot name="afterOptions" selection={$selection} />
</div>

<div class="grid grid-cols-[auto,1fr,auto] border-t border-surface-content/10 px-4 py-2">
  <slot name="actions" selection={$selection} {searchText}>
    <div />
  </slot>

  <div />

  <div>
    <Button
      class="px-6"
      disabled={applying}
      on:click={() => {
        $selection.reset();
        dispatch('cancel');
      }}
      {...cancelButtonProps}
    >
      Cancel
    </Button>

    <Button
      variant="fill"
      color="primary"
      class="px-6"
      loading={applying}
      disabled={!$isSelectionDirty || applying}
      on:click={async () => {
        applying = true;
        const changeContext = {
          value: $selection.selected,
          selection: $selection,
          indeterminate: $indeterminateStore,
          original: { selected: selectedOptions, unselected: unselectedOptions },
        };
        await onApply(changeContext);
        applying = false;
        onChange();
      }}
      {...applyButtonProps}
    >
      Apply
    </Button>
  </div>
</div>
