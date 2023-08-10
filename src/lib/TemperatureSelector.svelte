<script>
  import { createEventDispatcher, onMount } from "svelte";
  import Temperature from "./Temperature.svelte";
  import TemperatureDeviation from "./TemperatureDeviation.svelte";

  export let id;
  export let celsius;
  export let deviation;
  export let scale;
  let temperature = 0;
  let allowedDeviation = 0;

  const dispatch = createEventDispatcher();

  $: celsius, scale, setTemperature();

  onMount(() => {
    setTemperature();
  });

  // Defaults for Celsius
  let rangeMin = 10;
  let rangeMax = 30;
  let step = 0.5;
  let deviationMin = 0;
  let deviationMax = 5;
  let deviationStep = 0.5;

  // Convert celcius to the desired scale...
  function setTemperature() {
    switch (scale) {
      case "celsius":
        temperature = celsius;
        allowedDeviation = deviation;
        rangeMin = 10;
        rangeMax = 30;
        deviationMin = 0;
        deviationMax = 5;
        break;
      case "fahrenheit":
        temperature = celsius * (9 / 5) + 32;
        allowedDeviation = deviation * (9 / 5);
        rangeMin = 50;
        rangeMax = 86;
        deviationMin = 0;
        deviationMax = 9;
        break;
      case "kelvin":
        temperature = celsius + 273.15;
        allowedDeviation = deviation;
        rangeMin = 283;
        rangeMax = 303;
        deviationMin = 0;
        deviationMax = 5;
        break;
    }
  }

  function calculateTemperature() {
    switch (scale) {
      case "celsius":
        celsius = temperature;
        deviation = allowedDeviation;
        break;
      case "fahrenheit":
        celsius = ((temperature - 32) * 5) / 9;
        deviation = (allowedDeviation * 5) / 9;
        break;
      case "kelvin":
        celsius = temperature - 273.15;
        deviation = allowedDeviation;
        break;
    }
    dispatchChange();
  }

  function dispatchChange() {
    dispatch("temperature", {
      value: celsius,
    });
  }
</script>

<div>
  <input
    {id}
    type="range"
    class="form-range"
    min={rangeMin}
    max={rangeMax}
    {step}
    aria-describedby="{id}Help"
    bind:value={temperature}
    on:input={calculateTemperature}
    on:change={calculateTemperature}
  />
  <div class="row text-center" id="{id}Help">
    <div class="col">
      <div class="form-text mt-0">
        Temperature <Temperature {celsius} {scale} />
      </div>
    </div>
  </div>
  <input
    type="range"
    class="form-range"
    min={deviationMin}
    max={deviationMax}
    step={deviationStep}
    id="{id}AllowedDeviation"
    bind:value={allowedDeviation}
    on:input={calculateTemperature}
    on:change={calculateTemperature}
    aria-describedby="{id}AllowedDeviationHelp"
  />
  <div class="row text-center" id="{id}AllowedDeviationHelp">
    <div class="col">
      <div class="form-text mt-0">
        Temperature allowed deviation ± <TemperatureDeviation
          celsius={deviation}
          {scale}
        />
      </div>
    </div>
  </div>
</div>
