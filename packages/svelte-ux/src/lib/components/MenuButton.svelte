<script lang="ts">
  import { createEventDispatcher } from 'svelte';
  import type { ComponentProps } from '$lib/types/index.js';
  import { getComponentSettings } from './settings.js';
  import { mdiMenuDown } from '@mdi/js';

  import { cls } from '../utils/styles.js';

  import Button from './Button.svelte';
  import Icon from './Icon.svelte';
  import Menu from './Menu.svelte';
  import MenuItem from './MenuItem.svelte';

  const dispatch = createEventDispatcher<{ change: { value: any } }>();
  const { classes: settingsClasses, defaults } = getComponentSettings('MenuButton');

  export let options: Array<{ label: string; value: any; icon?: string }>;
  export let value: any = null;
  export let menuProps: ComponentProps<Menu> = { placement: 'bottom-start' };
  export let menuIcon: string | null = mdiMenuDown;
  $: selected = options?.find((x) => x.value === value);

  export let classes: {
    root?: string;
    label?: string;
    icon?: string;
  } = {};

  let open = false;

  $: restProps = { ...defaults, ...$$restProps };
</script>

<Button
  on:click={() => (open = !open)}
  {...restProps}
  class={cls('MenuButton', settingsClasses.root, classes.root, $$props.class)}
>
  <slot name="selection">
    <span class={cls('truncate', settingsClasses.label, classes.label)}>
      {selected?.label ?? 'No selection'}
    </span>
  </slot>

  {#if menuIcon}
    <Icon
      path={menuIcon}
      class={cls(
        'opacity-50 transform transition-all -mr-2 duration-300',
        open && '-rotate-180',
        settingsClasses.icon,
        classes.icon
      )}
    />
  {/if}

  <Menu
    {open}
    on:close={() => {
      open = false;
    }}
    {...menuProps}
  >
    <slot {options} {selected} close={() => (open = false)} setValue={(val) => (value = val)}>
      <menu class="group p-1">
        {#each options as option}
          <MenuItem
            icon={option.icon}
            selected={option.value === value}
            on:click={() => {
              value = option.value;
              dispatch('change', { value });
            }}
          >
            {option.label}
          </MenuItem>
        {/each}
      </menu>
    </slot>
  </Menu>
</Button>
