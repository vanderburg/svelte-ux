<script lang="ts">
  import { createEventDispatcher, type ComponentProps } from 'svelte';
  import { slide } from 'svelte/transition';
  import type { TransitionConfig } from 'svelte/transition';
  import type { Placement } from '@floating-ui/dom';

  import { focusMove } from '../actions/focus.js';
  import { cls } from '../utils/styles.js';

  import Popover from './Popover.svelte';
  import type { TransitionParams } from '$lib/types/index.js';
  import { getComponentClasses } from './theme.js';

  const dispatch = createEventDispatcher();

  export let open = false;
  export let offset = 4;

  export let matchWidth: boolean = false;
  export let placement: Placement = matchWidth ? 'bottom-start' : 'bottom';
  export let autoPlacement = false;
  export let resize: ComponentProps<Popover>['resize'] = false;
  export let disableTransition = false;
  export let transition = disableTransition
    ? (node: HTMLElement, params: TransitionParams) => ({}) as TransitionConfig
    : slide;
  export let transitionParams: TransitionParams = {};
  export let explicitClose = false;
  export let moveFocus = true;

  export let classes: {
    root?: string;
    menu?: string;
  } = {};
  const settingsClasses = getComponentClasses('Menu');

  export let menuItemsEl: HTMLMenuElement | undefined = undefined;

  function onClick(e: MouseEvent) {
    try {
      if (e.target === menuItemsEl) {
        // Clicked within menu but outside of any items
        // console.log('clicked:menuItems', e.target, menuEl);
      } else if (!explicitClose) {
        //
        // console.log('clicked:menuItem', e.target);
        open = false;
        dispatch('close', 'item');
      }
    } catch (err) {
      console.error(err);
    }
  }
</script>

<Popover
  {placement}
  {autoPlacement}
  {offset}
  {matchWidth}
  {resize}
  {open}
  class={cls(
    'Menu',
    'bg-surface-100 rounded shadow border overflow-auto',
    settingsClasses.root,
    classes.root,
    $$props.class
  )}
  style={$$props.style}
  on:close
  let:close
>
  <!-- svelte-ignore a11y-click-events-have-key-events -->
  <menu
    class={cls('menu-items outline-none max-h-screen', settingsClasses.menu, classes.menu)}
    bind:this={menuItemsEl}
    on:click={onClick}
    on:mouseup={(e) => {
      // Do not allow event to reach Popover's on:mouseup (clickOutside)
      e.stopPropagation();
    }}
    transition:transition={transitionParams}
    use:focusMove={{ disabled: !moveFocus }}
  >
    <slot {close} />
  </menu>
</Popover>
