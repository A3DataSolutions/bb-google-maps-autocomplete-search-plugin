<script>
  import "@spectrum-css/inputgroup/dist/index-vars.css";
  import "@spectrum-css/popover/dist/index-vars.css";
  import "@spectrum-css/menu/dist/index-vars.css";
  import { createEventDispatcher } from "svelte";
  import clickOutside from "./click_outside.js";
  import Popover from "./Popover.svelte";

  export let value;
  export let id = null;
  export let placeholder;
  export let disabled = false;
  export let readonly = false;
  export let options = [];
  export let getOptionLabel = (option) => option;
  export let getOptionValue = (option) => option;
  export let no_values_msg = "No Values Found";
  let timer = null;

  const dispatch = createEventDispatcher();

  export let open = false;
  let focus = false;
  let anchor;

  //Can probably be removed
  // $: open = check_if_value(value)

  // function check_if_value(value){
  //   return !(value == null || value.trim() === '');
  // }

  const selectOption = (value) => {
    dispatch("change", value);
    open = false;
  };

  const onType = (e) => {
    const value = e.target.value;
    dispatch("type", value);
    selectOption(value);
  };

  const onInput = (e) => {
    const value = e.target.value;
    dispatch("input", value);
    dispatch("change", value);
    open = true;
  };

  const onPick = (value, option) => {
    dispatch("pick", value);
    dispatch("setPlaceID", value);
    dispatch("ady_selection");
    selectOption(option.label);
  };
</script>

<div
  class="spectrum-InputGroup"
  class:is-focused={open || focus}
  class:is-disabled={disabled}
  bind:this={anchor}
>
  <div
    class="spectrum-Textfield spectrum-InputGroup-textfield"
    class:is-disabled={disabled}
    class:is-focused={open || focus}
  >
    <input
      {id}
      type="text"
      on:focus={() => ((focus = true), dispatch("setUUID"))}
      on:blur={() => {
        focus = false;
        dispatch("blur",value);
      }}
      on:input={onInput}
      value={value || ""}
      placeholder={placeholder || ""}
      {disabled}
      {readonly}
      autocomplete="off"
      class="spectrum-Textfield-input spectrum-InputGroup-input"
    />
  </div>

  <button
    class="spectrum-Picker spectrum-Picker--sizeM spectrum-InputGroup-button"
    tabindex="-1"
    aria-haspopup="true"
    {disabled}
    on:click={() => (open = !open)}
  >
    <svg
      class="spectrum-Icon spectrum-UIIcon-ChevronDown100 spectrum-Picker-menuIcon spectrum-InputGroup-icon"
      focusable="false"
      aria-hidden="true"
    >
      <use xlink:href="#spectrum-css-icon-Chevron100" />
    </svg>
  </button>
</div>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<Popover
  {anchor}
  {open}
  align="left"
  on:close={() => (open = false)}
  useAnchorWidth
>
  <div class="popover-content" use:clickOutside={() => (open = false)}>
    <ul class="spectrum-Menu" role="listbox">
      {#if options && options.length != 0 && getOptionValue(options[0]) != "No-Addresses-Found" && Array.isArray(options)}
        {#each options as option}
          <li
            class="spectrum-Menu-item"
            class:is-selected={getOptionValue(option) === value}
            role="option"
            aria-selected="true"
            tabindex="0"
            on:click={() => onPick(getOptionValue(option), option)}
          >
            <span class="spectrum-Menu-itemLabel">{getOptionLabel(option)}</span
            >
            <svg
              class="spectrum-Icon spectrum-UIIcon-Checkmark100 spectrum-Menu-checkmark spectrum-Menu-itemIcon"
              focusable="false"
              aria-hidden="true"
            >
              <use xlink:href="#spectrum-css-icon-Checkmark100" />
            </svg>
          </li>
        {/each}
      {:else if getOptionValue(options[0]) == "No-Addresses-Found"}
        <li
          class="spectrum-Menu-item"
          class:is-disabled={getOptionValue(getOptionValue(options[0])) ===
            "No-Addresses-Found"}
          role="option"
          aria-selected="true"
          tabindex="0"
        >
          <span class="spectrum-Menu-itemLabel">{no_values_msg}</span>
          <svg
            class="spectrum-Icon spectrum-UIIcon-Checkmark100 spectrum-Menu-checkmark spectrum-Menu-itemIcon"
            focusable="false"
            aria-hidden="true"
          >
            <use xlink:href="#spectrum-css-icon-Checkmark100" />
          </svg>
        </li>
      {:else}
        <li
          class="spectrum-Menu-item is-disabled"
          role="option"
          aria-selected="true"
          tabindex="0"
        >
          <span class="spectrum-Menu-itemLabel">
            <div
              class="spectrum-ProgressCircle spectrum-ProgressCircle--indeterminate spectrum-ProgressCircle--small spectrum-Combobox-progress-circle"
            >
              <div class="spectrum-ProgressCircle-track"></div>
              <div class="spectrum-ProgressCircle-fills">
                <div class="spectrum-ProgressCircle-fillMask1">
                  <div class="spectrum-ProgressCircle-fillSubMask1">
                    <div class="spectrum-ProgressCircle-fill"></div>
                  </div>
                </div>
                <div class="spectrum-ProgressCircle-fillMask2">
                  <div class="spectrum-ProgressCircle-fillSubMask2">
                    <div class="spectrum-ProgressCircle-fill"></div>
                  </div>
                </div>
              </div>
            </div>
            Currently Loading
          </span>
        </li>
      {/if}
    </ul>
  </div>
</Popover>

<style>
  .spectrum-InputGroup {
    min-width: 0;
    width: 100%;
  }
  .spectrum-Textfield {
    width: 100%;
  }
  .spectrum-Textfield-input {
    width: 0;
  }

  /* Popover */
  .popover-content {
    display: contents;
  }
  .popover-content:not(.auto-width) .spectrum-Menu-itemLabel {
    width: 0;
    flex: 1 1 auto;
  }
</style>
