<script>
  import { getContext, onMount, onDestroy,createEventDispatcher } from "svelte";
  import Combobox from "./lib/Core/Combobox.svelte"
  import Field from "./lib/Core/Field.svelte"
  export let autocomplete_dp;
  export let key_column;
  export let label_column;
  export let field;
  export let label;
  export let disabled = false
  export let readonly = false
  export let labelPosition = "above"
  export let error = null
  export let placeholder;
  export let options;
  export let helpText = null
  export let getOptionLabel = option => extractProperty(option, "label")
  export let getOptionValue = option => extractProperty(option, "value")
  export let ady_selection;
  export let on_change_event;
  export let on_blur_event;
  export let no_values_msg;

  let fieldApi;
  let fieldState;

  const formContext = getContext("form");
  const formStepContext = getContext("form-step");

  function uuidv4() {
    return "10000000-1000-4000-8000-100000000000".replace(/[018]/g, (c) =>
      (
        +c ^
        (crypto.getRandomValues(new Uint8Array(1))[0] & (15 >> (+c / 4)))
      ).toString(16),
    );
  }

  const formApi = formContext?.formApi;
  $: formStep = formStepContext ? $formStepContext || 1 : 1;
  $: formField = formApi?.registerField(
    field,
    "text",
    "",
    false,
    null,
    formStep,
  );
  
  $: unsubscribe = formField?.subscribe((value) => {
    fieldState = value?.fieldState;
    fieldApi = value?.fieldApi;
  });

  onDestroy(() => {
    fieldApi?.deregister();
    unsubscribe?.();
  });

  export let value
  export let sessiontoken
  export let place_id

  $: dataContext = {
    value,
    sessiontoken,
    place_id
  }
  

  const { styleable, Provider } = getContext("sdk");
  const component = getContext("component");

  $: a = clean_dp(autocomplete_dp?.rows ?? [],autocomplete_dp);
 

  function clean_dp(dataprovider,testing) {
    options = []
    for (let i = 0, len = dataprovider.length; i < len; i++) {
          options.push({'value':dataprovider[i][key_column],'label':dataprovider[i][label_column]});
    }
    return options;
  }

  const dispatch = createEventDispatcher()


  const handleSelection = e => {
    if (ady_selection) {
      ady_selection({ place_id: place_id, sessiontoken: sessiontoken})
    }
  }

  const onChange = e => {
    value = e.detail
    dispatch("change", e.detail)
    if (on_change_event){
      on_change_event({ place_id: place_id, sessiontoken: sessiontoken})
    }
  }

  const onBlur = async e => {
    console.log(e.detail)
    await new Promise(resolve => setTimeout(resolve, 500)); //SMALL DELAY TO ASSURE ON CHANGE RUNS BEFORE ON BLUR
    if (on_blur_event){
      on_blur_event({ place_id: e.detail, sessiontoken: sessiontoken})
    }
  }

  const setUUID = e => {
    sessiontoken = uuidv4()
    focus = true
  }

  const setPlaceID = e => {
    place_id = e.detail
  }

  const extractProperty = (value, property) => {
    if (value && typeof value === "object") {
      return value[property]
    }
    return value
  }
</script>

<div use:styleable={$component.styles}>
  {#if !formContext}
    <div class="placeholder">Form components need to be wrapped in a form</div>
  {:else}
    <!-- {JSON.stringify(a)} -->
    <Provider data={dataContext}>
      <Field {helpText} {label} {labelPosition} {error}>
        <Combobox
          {error}
          {disabled}
          {value}
          {options}
          {placeholder}
          {readonly}
          {getOptionLabel}
          {getOptionValue}
          on:change={onChange}
          on:pick
          on:type
          on:blur={onBlur}
          on:setUUID={setUUID}
          on:ady_selection={handleSelection}
          on:setPlaceID={setPlaceID}
          {no_values_msg}
        />
      </Field>
      <slot />
    </Provider>
  {/if}
</div>

<style>
  
</style>