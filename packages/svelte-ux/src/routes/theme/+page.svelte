<script lang="ts">
  import { mdiChevronDown, mdiPalette, mdiWeatherNight, mdiWhiteBalanceSunny } from '@mdi/js';

  import Button from '$lib/components/Button.svelte';
  import ButtonGroup from '$lib/components/ButtonGroup.svelte';
  import CopyButton from '$lib/components/CopyButton.svelte';
  import Icon from '$lib/components/Icon.svelte';
  import Menu from '$lib/components/Menu.svelte';
  import MenuItem from '$lib/components/MenuItem.svelte';
  import MenuField from '$lib/components/MenuField.svelte';
  import SelectField from '$lib/components/SelectField.svelte';
  import Switch from '$lib/components/Switch.svelte';
  import Toggle from '$lib/components/Toggle.svelte';
  import Tooltip from '$lib/components/Tooltip.svelte';

  import { styleProps } from '$lib/actions/styleProps.js';
  import {
    getThemeNames,
    processThemeColors,
    themeStylesString,
    type SupportedColorSpace,
  } from '$lib/styles/theme.js';
  import type { MenuOption } from '$lib/types/index.js';
  import ColorField from './ColorField.svelte';
  import { getSettings } from '$lib/components/settings.js';

  export let data;

  type ThemeMenuOption = MenuOption & {
    themeName: string;
    theme: typeof customLightTheme | typeof data.themes.daisy | typeof data.themes.skeleton;
  };

  const { currentTheme } = getSettings();

  let selectedLightThemeValue: string;
  let selectedDarkThemeValue: string;

  let showOptionalColors = false;

  let colorSpace: SupportedColorSpace = 'hsl';

  // Needs to always match app since using tailwind classes
  const themeColorSpace: SupportedColorSpace = 'hsl';

  const daisyThemeNames = getThemeNames(data.themes.daisy);
  const skeletonThemeNames = getThemeNames(data.themes.skeleton);

  let customLightTheme: Record<string, string> = {};
  let customDarkTheme: Record<string, string> = {};

  $: lightThemes = [
    {
      label: 'Custom',
      value: 'custom',
      themeName: 'custom',
      theme: customLightTheme,
    },
    ...daisyThemeNames.light.map((themeName) => {
      return {
        label: themeName === 'light' ? 'light (daisy)' : themeName,
        value: themeName === 'light' ? 'daisy-light' : themeName,
        group: 'Daisy',
        themeName,
        theme: data.themes.daisy[themeName],
      };
    }),
    ...skeletonThemeNames.light.map((themeName) => {
      return {
        label: themeName === 'light' ? 'light (skeleton)' : themeName,
        value: themeName === 'light' ? 'skeleton-light' : themeName,
        group: 'Skeleton',
        themeName,
        theme: data.themes.skeleton[themeName],
      };
    }),
  ] as ThemeMenuOption[];

  $: darkThemes = [
    {
      label: 'Custom',
      value: 'custom',
      themeName: 'custom',
      theme: customDarkTheme,
    },
    ...daisyThemeNames.dark.map((themeName) => {
      return {
        label: themeName === 'dark' ? 'dark (daisy)' : themeName,
        value: themeName === 'dark' ? 'daisy-dark' : themeName,
        group: 'Daisy',
        themeName,
        theme: data.themes.daisy[themeName],
      };
    }),
    ...skeletonThemeNames.dark.map((themeName) => {
      return {
        label: themeName === 'dark' ? 'dark (skeleton)' : themeName,
        value: themeName === 'dark' ? 'skeleton-dark' : themeName,
        group: 'Skeleton',
        themeName,
        theme: data.themes.skeleton[themeName],
      };
    }),
  ] as ThemeMenuOption[];

  // Set initial theme selections (skip custom)
  $: if (selectedLightThemeValue === undefined) {
    selectedLightThemeValue = lightThemes[1].value;
  }
  $: if (selectedDarkThemeValue === undefined) {
    selectedDarkThemeValue = darkThemes[1].value;
  }

  $: showDarkTheme = $currentTheme.dark;
  $: selectedLightTheme = lightThemes.find((d) => d.value === selectedLightThemeValue)?.theme;
  $: selectedDarkTheme = darkThemes.find((d) => d.value === selectedDarkThemeValue)?.theme;
  $: previewTheme = showDarkTheme ? selectedDarkTheme : selectedLightTheme;

  // Update site dark/light mode with preview for better experience (previewing and applying)
  $: currentTheme.setTheme(showDarkTheme ? 'dark' : 'light');

  // Break cyclical dependency with lightThemes => customLightTheme -> selectedLightTheme -> lightThemes
  function updateCustomLightTheme() {
    customLightTheme = {
      ...selectedLightTheme,
      primary: selectedLightTheme?.primary ?? selectedLightTheme?.['primary-500'],
      secondary: selectedLightTheme?.secondary ?? selectedLightTheme?.['secondary-500'],
      accent: selectedLightTheme?.accent ?? selectedLightTheme?.['accent-500'],
      neutral: selectedLightTheme?.neutral ?? selectedLightTheme?.['neutral-500'],
    };
  }
  $: selectedLightTheme && updateCustomLightTheme();

  // Break cyclical dependency with darkThemes => customDarkTheme -> selectedDarkTheme -> darkThemes
  function updateCustomLDarkTheme() {
    customDarkTheme = {
      ...selectedDarkTheme,
      primary: selectedDarkTheme?.primary ?? selectedDarkTheme?.['primary-500'],
      secondary: selectedDarkTheme?.secondary ?? selectedDarkTheme?.['secondary-500'],
      accent: selectedDarkTheme?.accent ?? selectedDarkTheme?.['accent-500'],
      neutral: selectedDarkTheme?.neutral ?? selectedDarkTheme?.['neutral-500'],
    };
  }
  $: selectedDarkTheme && updateCustomLDarkTheme();
</script>

<main class="p-4 grid gap-5">
  <div class="grid sm:grid-cols-[1fr,1fr,auto,auto] gap-3">
    <SelectField
      label="Light theme"
      options={lightThemes}
      bind:value={selectedLightThemeValue}
      clearable={false}
      toggleIcon={null}
      stepper
      on:change={() => (showDarkTheme = false)}
    />
    <SelectField
      label="Dark theme"
      options={darkThemes}
      bind:value={selectedDarkThemeValue}
      clearable={false}
      toggleIcon={null}
      stepper
      on:change={() => (showDarkTheme = true)}
    />

    <MenuField
      label="Color space"
      bind:value={colorSpace}
      options={[
        { label: 'hex', value: 'hex' },
        { label: 'hsl', value: 'hsl' },
        { label: 'rgb', value: 'rgb' },
        { label: 'oklch', value: 'oklch' },
      ]}
    />

    <ButtonGroup variant="fill" color="primary">
      <Tooltip title="Copy themes to clipboard" offset={2}>
        <CopyButton
          value={JSON.stringify({ light: selectedLightTheme, dark: selectedDarkTheme }, null, 2)}
        />
      </Tooltip>

      <Tooltip title="Override current themes" offset={2}>
        <Button
          icon={mdiPalette}
          iconOnly={false}
          on:click={() => {
            const style =
              document.getElementById('custom-theme') ?? document.createElement('style');
            style.id = 'custom-theme';

            style.textContent = `
              :root { ${themeStylesString(selectedLightTheme, themeColorSpace)} }
              @media (prefers-color-scheme: dark) {
                :root {
                  ${themeStylesString(selectedDarkTheme, themeColorSpace)}
                }
              }
              [data-theme=light] { ${themeStylesString(selectedLightTheme, themeColorSpace)} }
              [data-theme=dark] { ${themeStylesString(selectedDarkTheme, themeColorSpace)} }
            `;
            document.head.appendChild(style);

            // TODO: Update settings({ themes: { light: ['light'], dark: ['dark'] }})
          }}
        />
      </Tooltip>

      <Toggle let:on={open} let:toggle>
        <div class="grid">
          <Button icon={mdiChevronDown} on:click={toggle} rounded class="px-1" />
          <Menu {open} on:close={toggle} placement="bottom-start">
            <MenuItem
              on:click={() => {
                const allThemes = {
                  ...data.themes.daisy,
                  ...Object.fromEntries(
                    Object.entries(data.themes.skeleton).map(([themeName, value]) => {
                      return [
                        themeName === 'light'
                          ? 'skeleton-light'
                          : themeName === 'dark'
                            ? 'skeleton-dark'
                            : themeName,
                        value,
                      ];
                    })
                  ),
                };
                const clipboardData = JSON.stringify(allThemes, null, 2);
                navigator.clipboard.writeText(clipboardData);
              }}
            >
              Copy All themes
            </MenuItem>
            <MenuItem
              on:click={() => {
                const clipboardData = JSON.stringify(data.themes.daisy, null, 2);
                navigator.clipboard.writeText(clipboardData);
              }}
            >
              Copy Daisy themes
            </MenuItem>
            <MenuItem
              on:click={() => {
                const clipboardData = JSON.stringify(data.themes.skeleton, null, 2);
                navigator.clipboard.writeText(clipboardData);
              }}
            >
              Copy Skeleton themes
            </MenuItem>
          </Menu>
        </div>
      </Toggle>
    </ButtonGroup>
  </div>

  <div class="grid sm:grid-cols-2 gap-2">
    <div class="grid gap-2">
      <div class="text-xs -mb-1 text-surface-content/50 font-semibold">Light theme</div>
      <ColorField
        label="Primary"
        bind:value={customLightTheme.primary}
        on:change={() => {
          selectedLightThemeValue = 'custom';
          showDarkTheme = false;
        }}
        {colorSpace}
      />
      <ColorField
        label="Secondary"
        bind:value={customLightTheme.secondary}
        on:change={() => {
          selectedLightThemeValue = 'custom';
          showDarkTheme = false;
        }}
        {colorSpace}
      />
      <ColorField
        label="Accent"
        bind:value={customLightTheme.accent}
        on:change={() => {
          selectedLightThemeValue = 'custom';
          showDarkTheme = false;
        }}
        {colorSpace}
      />

      {#if showOptionalColors}
        <ColorField
          label="Neutral (optional)"
          bind:value={customLightTheme.neutral}
          on:change={() => {
            selectedLightThemeValue = 'custom';
            showDarkTheme = false;
          }}
          {colorSpace}
        />
        <!-- State colors -->
        <ColorField
          label="Info (optional)"
          bind:value={customLightTheme.info}
          on:change={() => {
            selectedLightThemeValue = 'custom';
            showDarkTheme = false;
          }}
          {colorSpace}
        />
        <ColorField
          label="Success (optional)"
          bind:value={customLightTheme.success}
          on:change={() => {
            selectedLightThemeValue = 'custom';
            showDarkTheme = false;
          }}
          {colorSpace}
        />
        <ColorField
          label="Warning (optional)"
          bind:value={customLightTheme.warning}
          on:change={() => {
            selectedLightThemeValue = 'custom';
            showDarkTheme = false;
          }}
          {colorSpace}
        />
        <ColorField
          label="Danger (optional)"
          bind:value={customLightTheme.danger}
          on:change={() => {
            selectedLightThemeValue = 'custom';
            showDarkTheme = false;
          }}
          {colorSpace}
        />
      {/if}

      <ColorField
        label="Surface 100"
        bind:value={customLightTheme['surface-100']}
        on:change={() => {
          selectedLightThemeValue = 'custom';
          showDarkTheme = false;
        }}
        {colorSpace}
      />
      {#if showOptionalColors}
        <ColorField
          label="Surface 200 (optional)"
          bind:value={customLightTheme['surface-200']}
          on:change={() => {
            selectedLightThemeValue = 'custom';
            showDarkTheme = false;
          }}
          {colorSpace}
        />
        <ColorField
          label="Surface 300 (optional)"
          bind:value={customLightTheme['surface-300']}
          on:change={() => {
            selectedLightThemeValue = 'custom';
            showDarkTheme = false;
          }}
          {colorSpace}
        />
      {/if}
    </div>

    <div class="grid gap-2">
      <div class="text-xs -mb-1 text-surface-content/50 font-semibold">Dark theme</div>
      <ColorField
        label="Primary"
        bind:value={customDarkTheme.primary}
        on:change={() => {
          selectedDarkThemeValue = 'custom';
          showDarkTheme = true;
        }}
        {colorSpace}
      />
      <ColorField
        label="Secondary"
        bind:value={customDarkTheme.secondary}
        on:change={() => {
          selectedDarkThemeValue = 'custom';
          showDarkTheme = true;
        }}
        {colorSpace}
      />
      <ColorField
        label="Accent"
        bind:value={customDarkTheme.accent}
        on:change={() => {
          selectedDarkThemeValue = 'custom';
          showDarkTheme = true;
        }}
        {colorSpace}
      />

      {#if showOptionalColors}
        <ColorField
          label="Neutral (optional)"
          bind:value={customDarkTheme.neutral}
          on:change={() => {
            selectedDarkThemeValue = 'custom';
            showDarkTheme = true;
          }}
          {colorSpace}
        />
        <!-- State colors -->
        <ColorField
          label="Info (optional)"
          bind:value={customDarkTheme.info}
          on:change={() => {
            selectedDarkThemeValue = 'custom';
            showDarkTheme = true;
          }}
          {colorSpace}
        />
        <ColorField
          label="Success (optional)"
          bind:value={customDarkTheme.success}
          on:change={() => {
            selectedDarkThemeValue = 'custom';
            showDarkTheme = true;
          }}
          {colorSpace}
        />
        <ColorField
          label="Warning (optional)"
          bind:value={customDarkTheme.warning}
          on:change={() => {
            selectedDarkThemeValue = 'custom';
            showDarkTheme = true;
          }}
          {colorSpace}
        />
        <ColorField
          label="Danger (optional)"
          bind:value={customDarkTheme.danger}
          on:change={() => {
            selectedDarkThemeValue = 'custom';
            showDarkTheme = true;
          }}
          {colorSpace}
        />
      {/if}

      <ColorField
        label="Surface 100"
        bind:value={customDarkTheme['surface-100']}
        on:change={() => {
          selectedDarkThemeValue = 'custom';
          showDarkTheme = true;
        }}
        {colorSpace}
      />
      {#if showOptionalColors}
        <ColorField
          label="Surface 200 (optional)"
          bind:value={customDarkTheme['surface-200']}
          on:change={() => {
            selectedDarkThemeValue = 'custom';
            showDarkTheme = true;
          }}
          {colorSpace}
        />
        <ColorField
          label="Surface 300 (optional)"
          bind:value={customDarkTheme['surface-300']}
          on:change={() => {
            selectedDarkThemeValue = 'custom';
            showDarkTheme = true;
          }}
          {colorSpace}
        />
      {/if}
    </div>

    <div>
      <Button color="primary" size="sm" on:click={() => (showOptionalColors = !showOptionalColors)}>
        {showOptionalColors ? 'Hide' : 'Show'} optional colors
      </Button>
    </div>
  </div>

  <div>
    <div class="grid grid-cols-[1fr,auto] border-b pb-1 mb-2 font-semibold text-xl">
      Theme preview

      <Switch
        checked={showDarkTheme}
        on:change={(e) => (showDarkTheme = e.target.checked)}
        let:checked
      >
        {#if checked}
          <Icon data={mdiWeatherNight} size=".8rem" class="text-primary" />
        {:else}
          <Icon data={mdiWhiteBalanceSunny} size=".8rem" class="text-primary" />
        {/if}
      </Switch>
    </div>

    <div
      class="bg-surface-100 border rounded py-2 px-3"
      style:color={previewTheme?.['color-scheme'] === 'dark' ? 'white' : 'black'}
      use:styleProps={previewTheme ? processThemeColors(previewTheme, 'hsl') : {}}
    >
      <div>
        <div class="text-xl font-bold mb-2">Semantic colors</div>

        <div class="grid grid-cols-xs gap-3">
          <div class="swatch">
            <div class="bg-primary text-primary-content name">Primary</div>
            <div class="bg-primary-50 shade"><span>50</span></div>
            <div class="bg-primary-100 shade"><span>100</span></div>
            <div class="bg-primary-200 shade"><span>200</span></div>
            <div class="bg-primary-300 shade"><span>300</span></div>
            <div class="bg-primary-400 shade"><span>400</span></div>
            <div class="bg-primary-500 shade"><span>500</span></div>
            <div class="bg-primary-600 shade"><span>600</span></div>
            <div class="bg-primary-700 shade"><span>700</span></div>
            <div class="bg-primary-800 shade"><span>800</span></div>
            <div class="bg-primary-900 shade"><span>900</span></div>
          </div>

          <div class="swatch">
            <div class="bg-secondary text-secondary-content name">Secondary</div>
            <div class="bg-secondary-50 shade"><span>50</span></div>
            <div class="bg-secondary-100 shade"><span>100</span></div>
            <div class="bg-secondary-200 shade"><span>200</span></div>
            <div class="bg-secondary-300 shade"><span>300</span></div>
            <div class="bg-secondary-400 shade"><span>400</span></div>
            <div class="bg-secondary-500 shade"><span>500</span></div>
            <div class="bg-secondary-600 shade"><span>600</span></div>
            <div class="bg-secondary-700 shade"><span>700</span></div>
            <div class="bg-secondary-800 shade"><span>800</span></div>
            <div class="bg-secondary-900 shade"><span>900</span></div>
          </div>

          <div class="swatch">
            <div class="bg-accent text-accent-content name">Accent</div>
            <div class="bg-accent-50 shade"><span>50</span></div>
            <div class="bg-accent-100 shade"><span>100</span></div>
            <div class="bg-accent-200 shade"><span>200</span></div>
            <div class="bg-accent-300 shade"><span>300</span></div>
            <div class="bg-accent-400 shade"><span>400</span></div>
            <div class="bg-accent-500 shade"><span>500</span></div>
            <div class="bg-accent-600 shade"><span>600</span></div>
            <div class="bg-accent-700 shade"><span>700</span></div>
            <div class="bg-accent-800 shade"><span>800</span></div>
            <div class="bg-accent-900 shade"><span>900</span></div>
          </div>

          <div class="swatch">
            <div class="bg-neutral text-neutral-content name">Neutral</div>
            <div class="bg-neutral-50 shade"><span>50</span></div>
            <div class="bg-neutral-100 shade"><span>100</span></div>
            <div class="bg-neutral-200 shade"><span>200</span></div>
            <div class="bg-neutral-300 shade"><span>300</span></div>
            <div class="bg-neutral-400 shade"><span>400</span></div>
            <div class="bg-neutral-500 shade"><span>500</span></div>
            <div class="bg-neutral-600 shade"><span>600</span></div>
            <div class="bg-neutral-700 shade"><span>700</span></div>
            <div class="bg-neutral-800 shade"><span>800</span></div>
            <div class="bg-neutral-900 shade"><span>900</span></div>
          </div>
        </div>
      </div>

      <div>
        <div class="text-xl font-bold mt-4 mb-2">State colors</div>

        <div class="grid grid-cols-xs gap-3">
          <div class="swatch">
            <div class="bg-info text-info-content name">Info</div>
            <div class="bg-info-50 shade"><span>50</span></div>
            <div class="bg-info-100 shade"><span>100</span></div>
            <div class="bg-info-200 shade"><span>200</span></div>
            <div class="bg-info-300 shade"><span>300</span></div>
            <div class="bg-info-400 shade"><span>400</span></div>
            <div class="bg-info-500 shade"><span>500</span></div>
            <div class="bg-info-600 shade"><span>600</span></div>
            <div class="bg-info-700 shade"><span>700</span></div>
            <div class="bg-info-800 shade"><span>800</span></div>
            <div class="bg-info-900 shade"><span>900</span></div>
          </div>

          <div class="swatch">
            <div class="bg-success text-success-content name">Success</div>
            <div class="bg-success-50 shade"><span>50</span></div>
            <div class="bg-success-100 shade"><span>100</span></div>
            <div class="bg-success-200 shade"><span>200</span></div>
            <div class="bg-success-300 shade"><span>300</span></div>
            <div class="bg-success-400 shade"><span>400</span></div>
            <div class="bg-success-500 shade"><span>500</span></div>
            <div class="bg-success-600 shade"><span>600</span></div>
            <div class="bg-success-700 shade"><span>700</span></div>
            <div class="bg-success-800 shade"><span>800</span></div>
            <div class="bg-success-900 shade"><span>900</span></div>
          </div>

          <div class="swatch">
            <div class="bg-warning text-warning-content name">Warning</div>
            <div class="bg-warning-50 shade"><span>50</span></div>
            <div class="bg-warning-100 shade"><span>100</span></div>
            <div class="bg-warning-200 shade"><span>200</span></div>
            <div class="bg-warning-300 shade"><span>300</span></div>
            <div class="bg-warning-400 shade"><span>400</span></div>
            <div class="bg-warning-500 shade"><span>500</span></div>
            <div class="bg-warning-600 shade"><span>600</span></div>
            <div class="bg-warning-700 shade"><span>700</span></div>
            <div class="bg-warning-800 shade"><span>800</span></div>
            <div class="bg-warning-900 shade"><span>900</span></div>
          </div>

          <div class="swatch">
            <div class="bg-danger text-danger-content name">Danger</div>
            <div class="bg-danger-50 shade"><span>50</span></div>
            <div class="bg-danger-100 shade"><span>100</span></div>
            <div class="bg-danger-200 shade"><span>200</span></div>
            <div class="bg-danger-300 shade"><span>300</span></div>
            <div class="bg-danger-400 shade"><span>400</span></div>
            <div class="bg-danger-500 shade"><span>500</span></div>
            <div class="bg-danger-600 shade"><span>600</span></div>
            <div class="bg-danger-700 shade"><span>700</span></div>
            <div class="bg-danger-800 shade"><span>800</span></div>
            <div class="bg-danger-900 shade"><span>900</span></div>
          </div>
        </div>
      </div>

      <div>
        <div class="text-xl font-bold mt-4 mb-2">Surface colors</div>

        <div class="grid grid-cols-xs gap-3">
          <div class="swatch">
            <div class="bg-surface text-surface-content name">Surface</div>
            <div class="bg-surface-100 shade"><span>100</span></div>
            <div class="bg-surface-200 shade"><span>200</span></div>
            <div class="bg-surface-300 shade"><span>300</span></div>
          </div>
        </div>
      </div>
    </div>
  </div>
</main>

<style lang="postcss">
  .swatch {
    @apply rounded-lg overflow-hidden border;
  }
  .swatch .name {
    @apply grid place-items-center items-center font-semibold py-2;
  }
  .shade {
    @apply grid place-items-center py-1 text-sm;
  }
  .shade span {
    @apply bg-black/10 rounded px-2 text-white;
  }
</style>
