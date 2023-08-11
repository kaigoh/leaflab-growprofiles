<script lang="ts">
  import type { GrowProfile } from "./lib/GrowProfile.svelte";
  import { onMount } from "svelte";
  import TemperatureSelector from "./lib/TemperatureSelector.svelte";
  import QRCodeStyling from "qr-code-styling";

  let temperatureScales = ["celsius", "fahrenheit", "kelvin"];
  let temperatureScale = "celsius";
  let growProfile = <GrowProfile>{
    name: "My Grow Profile",
    lighting: true,
    lights_off: 720,
    minimum_intake: 0,
    maximum_intake: 80,
    minimum_exhaust: 20,
    maximum_exhaust: 100,
    temperature_day: 21,
    temperature_day_allowed_deviation: 0.5,
    temperature_night: 20,
    temperature_night_allowed_deviation: 0.5,
    relative_humidity_day: 60,
    relative_humidity_day_allowed_deviation: 5,
    relative_humidity_night: 55,
    relative_humidity_night_allowed_deviation: 5,
    vpd_day: 0,
    vpd_night: 0,
  };

  $: growProfile,
    generateQRCode(),
    storeProfile(),
    calculateVPD("day"),
    calculateVPD("night");

  let qrCode;

  onMount(() => {
    const gp = localStorage.getItem("_leaflab_grow_profile");
    if (gp != null) {
      growProfile = JSON.parse(gp);
    }
    const ts = localStorage.getItem("_leaflab_temperatureScale");
    if (ts != null) {
      temperatureScale = ts;
    }
  });

  function calculateVPD(dn) {
    const temperature =
      dn == "day" ? growProfile.temperature_day : growProfile.temperature_night;
    const humidity =
      dn == "day"
        ? growProfile.relative_humidity_day
        : growProfile.relative_humidity_night;
    const a = 7.5;
    const b = 237.3;
    const relativeHumidityFraction = humidity / 100.0;
    const es = 6.11 * Math.pow(10, (a * temperature) / (b + temperature));
    const ea = es * relativeHumidityFraction;
    const vpd = (es - ea) / 10;
    if (dn == "day") {
      growProfile.vpd_day = vpd;
    } else {
      growProfile.vpd_night = vpd;
    }
  }

  function storeProfile() {
    localStorage.setItem("_leaflab_grow_profile", JSON.stringify(growProfile));
  }

  let containerWidth = 80;

  $: containerWidth, generateQRCode();

  function generateQRCode() {
    const separator = "|";
    const encodedString = Object.values(growProfile).join(separator);

    qrCode = new QRCodeStyling({
      width: containerWidth * 0.9,
      height: containerWidth * 0.9,
      data: encodedString,
      margin: 0,
      qrOptions: {
        typeNumber: 0,
        mode: "Byte",
        errorCorrectionLevel: "Q",
      },
      imageOptions: { hideBackgroundDots: false, imageSize: 0.4, margin: 0 },
      dotsOptions: {
        type: "extra-rounded",
        color: "#6a1a4c",
        gradient: {
          type: "linear",
          rotation: 0,
          colorStops: [
            { offset: 0, color: "#03668c" },
            { offset: 1, color: "#49af8a" },
          ],
        },
      },
      backgroundOptions: { color: "#ffffff" },
      image: "/img/icon.png",
      dotsOptionsHelper: {
        colorType: { single: true, gradient: false },
        gradient: {
          linear: true,
          radial: false,
          color1: "#6a1a4c",
          color2: "#6a1a4c",
          rotation: "0",
        },
      },
      cornersSquareOptions: { type: "extra-rounded", color: "#49af8a" },
      cornersSquareOptionsHelper: {
        colorType: { single: true, gradient: false },
        gradient: {
          linear: true,
          radial: false,
          color1: "#000000",
          color2: "#000000",
          rotation: "0",
        },
      },
      cornersDotOptions: { type: "dot", color: "#03668c" },
      cornersDotOptionsHelper: {
        colorType: { single: true, gradient: false },
        gradient: {
          linear: true,
          radial: false,
          color1: "#000000",
          color2: "#000000",
          rotation: "0",
        },
      },
      backgroundOptionsHelper: {
        colorType: { single: true, gradient: false },
        gradient: {
          linear: true,
          radial: false,
          color1: "#ffffff",
          color2: "#ffffff",
          rotation: "0",
        },
      },
    });
    const elem = document.getElementById("qrcode");
    if (elem) {
      elem.innerHTML = "";
      qrCode.append(elem);
    }
  }

  function downloadQRCode() {
    qrCode.download({ name: growProfile.name, extension: "png" });
  }

  function capitalizeFirstLetter(string) {
    return string.charAt(0).toUpperCase() + string.slice(1);
  }
</script>

<div class="pt-3">
  <div class="row text-center mb-3">
    <div class="col-sm-8">
      <img src="img/logo.svg" class="img-fluid mx-auto d-block" />
    </div>
  </div>
  <div class="row text-center">
    <div class="col">
      <h4>LeafLab Grow Profile Generator</h4>
    </div>
  </div>
  <div class="container">
    <div class="row pt-3">
      <div class="col-sm-4 mb-3">
        <div class="card">
          <div class="card-body text-center" bind:offsetWidth={containerWidth}>
            <div id="qrcode" on:click={downloadQRCode} />
          </div>
        </div>
      </div>
      <div class="col-sm-8">
        <form>
          <div class="accordion" id="growProfileSettings">
            <div class="accordion-item">
              <h2 class="accordion-header">
                <button
                  class="accordion-button collapsed"
                  type="button"
                  data-bs-toggle="collapse"
                  data-bs-target="#collapseTemperatureScale"
                  aria-expanded="true"
                  aria-controls="collapseTemperatureScale"
                >
                  <div class="row">
                    <div class="col-1">
                      <i class="fa-solid fa-thermometer" />
                    </div>
                    <div class="col">Temperature Scale</div>
                  </div>
                </button>
              </h2>
              <div
                id="collapseTemperatureScale"
                class="accordion-collapse collapse"
                data-bs-parent="#growProfileSettings"
              >
                <div class="accordion-body">
                  <select
                    class="form-select"
                    id="$environmentTemperatureScale"
                    bind:value={temperatureScale}
                    on:change={() => {
                      localStorage.setItem(
                        "_leaflab_temperatureScale",
                        temperatureScale
                      );
                    }}
                  >
                    {#each temperatureScales as t}
                      <option value={t}>{capitalizeFirstLetter(t)}</option>
                    {/each}
                  </select>
                </div>
              </div>
            </div>

            <div class="accordion-item">
              <h2 class="accordion-header">
                <button
                  class="accordion-button"
                  type="button"
                  data-bs-toggle="collapse"
                  data-bs-target="#collapseName"
                  aria-expanded="true"
                  aria-controls="collapseName"
                >
                  <div class="row">
                    <div class="col-1">
                      <i class="fa-solid fa-pencil" />
                    </div>
                    <div class="col">Profile Name</div>
                  </div>
                </button>
              </h2>
              <div
                id="collapseName"
                class="accordion-collapse collapse show"
                data-bs-parent="#growProfileSettings"
              >
                <div class="accordion-body">
                  <input
                    type="text"
                    class="form-control"
                    id="profileName"
                    bind:value={growProfile.name}
                  />
                </div>
              </div>
            </div>

            <div class="accordion-item">
              <h2 class="accordion-header">
                <button
                  class="accordion-button collapsed"
                  type="button"
                  data-bs-toggle="collapse"
                  data-bs-target="#collapseLighting"
                  aria-expanded="true"
                  aria-controls="collapseLighting"
                >
                  <div class="row">
                    <div class="col-1">
                      <i
                        class="fa-solid fa-lightbulb {growProfile.lighting
                          ? 'text-warning'
                          : 'text-secondary'}"
                      />
                    </div>
                    <div class="col">Lighting</div>
                  </div>
                </button>
              </h2>
              <div
                id="collapseLighting"
                class="accordion-collapse collapse"
                data-bs-parent="#growProfileSettings"
              >
                <div class="accordion-body">
                  <div class="mb-2">
                    <div class="form-check">
                      <input
                        class="form-check-input"
                        type="checkbox"
                        bind:checked={growProfile.lighting}
                        id="profileLighting"
                      />
                      <label class="form-check-label" for="profileLighting">
                        Lighting Enabled
                      </label>
                    </div>
                  </div>

                  {#if growProfile.lighting}
                    <div class="mb-2">
                      <label for="profileLightingDuration" class="form-label"
                        >Lighting Duration
                      </label>
                      <input
                        type="range"
                        class="form-range"
                        min="30"
                        max="1440"
                        step="30"
                        id="profileLightingDuration"
                        bind:value={growProfile.lights_off}
                        aria-describedby="profileLightingDurationHelp"
                      />
                      <div
                        class="row text-center"
                        id="profileLightingDurationHelp"
                      >
                        <div class="col">
                          <div class="form-text mt-0">
                            On for {growProfile.lights_off / 60} Hours
                          </div>
                        </div>
                        <div class="col">
                          <div class="form-text mt-0">
                            Off for {24 - growProfile.lights_off / 60} Hours
                          </div>
                        </div>
                      </div>
                    </div>
                  {/if}
                </div>
              </div>
            </div>

            <div class="accordion-item">
              <h2 class="accordion-header">
                <button
                  class="accordion-button collapsed"
                  type="button"
                  data-bs-toggle="collapse"
                  data-bs-target="#collapseAirflow"
                  aria-expanded="true"
                  aria-controls="collapseAirflow"
                >
                  <div class="row">
                    <div class="col-1">
                      <i class="fa-solid fa-wind text-primary" />
                    </div>
                    <div class="col">Air Flow</div>
                  </div>
                </button>
              </h2>
              <div
                id="collapseAirflow"
                class="accordion-collapse collapse"
                data-bs-parent="#growProfileSettings"
              >
                <div class="accordion-body">
                  <div class="card mb-2">
                    <div class="card-body">
                      <h6 class="card-title">
                        <i class="fa-solid fa-wind" /><i
                          class="fa-regular fa-fan me-2"
                        />Intake
                      </h6>
                      <div class="mb-2">
                        <input
                          type="range"
                          class="form-range"
                          min="0"
                          max="100"
                          step="10"
                          id="profileIntake"
                          bind:value={growProfile.minimum_intake}
                          aria-describedby="profileIntakeHelp"
                        />
                        <div class="row text-center" id="profileMinIntakeHelp">
                          <div class="col">
                            <div class="form-text mt-0">
                              Minimum Intake {growProfile.minimum_intake}%
                            </div>
                          </div>
                        </div>

                        <input
                          type="range"
                          class="form-range"
                          min="0"
                          max="100"
                          step="10"
                          id="profileMinIntake"
                          bind:value={growProfile.maximum_intake}
                          aria-describedby="profileMaxIntakeHelp"
                        />
                        <div class="row text-center" id="profileMaxIntakeHelp">
                          <div class="col">
                            <div class="form-text mt-0">
                              Maximum Intake {growProfile.maximum_intake}%
                            </div>
                          </div>
                        </div>
                      </div>
                    </div>
                  </div>

                  <div class="card mb-2">
                    <div class="card-body">
                      <h6 class="card-title">
                        <i class="fa-solid fa-fan" /><i
                          class="fa-regular fa-wind me-2"
                        />Exhaust
                      </h6>
                      <div class="mb-2">
                        <input
                          type="range"
                          class="form-range"
                          min="10"
                          max="100"
                          step="10"
                          id="profileExhaust"
                          bind:value={growProfile.minimum_exhaust}
                          aria-describedby="profileMinExhaustHelp"
                        />
                        <div class="row text-center" id="profileMinExhaustHelp">
                          <div class="col">
                            <div class="form-text mt-0">
                              Minimum Exhaust {growProfile.minimum_exhaust}%
                            </div>
                          </div>
                        </div>

                        <input
                          type="range"
                          class="form-range"
                          min="10"
                          max="100"
                          step="10"
                          id="profileExhaust"
                          bind:value={growProfile.maximum_exhaust}
                          aria-describedby="profileMaxExhaustHelp"
                        />
                        <div class="row text-center" id="profileMaxExhaustHelp">
                          <div class="col">
                            <div class="form-text mt-0">
                              Maximum Exhaust {growProfile.maximum_exhaust}%
                            </div>
                          </div>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>

            <div class="accordion-item">
              <h2 class="accordion-header">
                <button
                  class="accordion-button collapsed no-accordion-icon"
                  type="button"
                  aria-expanded="true"
                  aria-controls="collapseTemperature"
                  disabled
                >
                  <div class="row">
                    <div class="col-1">
                      <i class="fa-solid fa-water text-primary" />
                    </div>
                    <div class="col">
                      VPD
                      {#if !growProfile.lighting}
                        {growProfile.vpd_night.toFixed(1)}kPa
                      {:else if growProfile.lights_off == 1440}
                        {growProfile.vpd_day.toFixed(1)}kPa
                      {:else}
                        <i
                          class="fa-solid fa-sun text-warning ms-1 me-2"
                        />{growProfile.vpd_day.toFixed(1)}kPa
                        <span class="ms-1 me-1" />
                        <i
                          class="fa-solid fa-moon text-secondary ms-2 me-2"
                        />{growProfile.vpd_night.toFixed(1)}kPa
                      {/if}
                    </div>
                  </div>
                </button>
              </h2>
              <div
                id="collapseVPD"
                class="accordion-collapse collapse"
                data-bs-parent="#growProfileSettings"
              >
                <div class="accordion-body" />
              </div>
            </div>

            <div class="accordion-item">
              <h2 class="accordion-header">
                <button
                  class="accordion-button collapsed"
                  type="button"
                  data-bs-toggle="collapse"
                  data-bs-target="#collapseTemperature"
                  aria-expanded="true"
                  aria-controls="collapseTemperature"
                >
                  <div class="row">
                    <div class="col-1">
                      <i
                        class="fa-regular fa-temperature-high text-secondary"
                      />
                    </div>
                    <div class="col">Temperature</div>
                  </div>
                </button>
              </h2>
              <div
                id="collapseTemperature"
                class="accordion-collapse collapse"
                data-bs-parent="#growProfileSettings"
              >
                <div class="accordion-body">
                  {#if !growProfile.lighting}
                    <div class="card">
                      <div class="card-body">
                        <div class="mb-2">
                          <TemperatureSelector
                            id="profileTemperatureNight"
                            scale={temperatureScale}
                            bind:celsius={growProfile.temperature_night}
                            bind:deviation={growProfile.temperature_night_allowed_deviation}
                            on:temperature={() => calculateVPD("night")}
                          />
                        </div>
                      </div>
                    </div>
                  {:else if growProfile.lights_off == 1440}
                    <div class="card mb-2">
                      <div class="card-body">
                        <div class="mb-2">
                          <TemperatureSelector
                            id="profileTemperatureDay"
                            scale={temperatureScale}
                            bind:celsius={growProfile.temperature_day}
                            bind:deviation={growProfile.temperature_day_allowed_deviation}
                            on:temperature={() => calculateVPD("day")}
                          />
                        </div>
                      </div>
                    </div>
                  {:else}
                    <div class="card mb-2">
                      <div class="card-body">
                        <h6 class="card-title">
                          <i class="fa-regular fa-sun text-warning me-2" />Day
                        </h6>
                        <div class="mb-2">
                          <TemperatureSelector
                            id="profileTemperatureDay"
                            scale={temperatureScale}
                            bind:celsius={growProfile.temperature_day}
                            bind:deviation={growProfile.temperature_day_allowed_deviation}
                            on:temperature={() => calculateVPD("day")}
                          />
                        </div>
                      </div>
                    </div>
                    <div class="card">
                      <div class="card-body">
                        <h6 class="card-title">
                          <i class="fa-regular fa-moon me-2" />Night
                        </h6>
                        <div class="mb-2">
                          <TemperatureSelector
                            id="profileTemperatureNight"
                            scale={temperatureScale}
                            bind:celsius={growProfile.temperature_night}
                            bind:deviation={growProfile.temperature_night_allowed_deviation}
                            on:temperature={() => calculateVPD("night")}
                          />
                        </div>
                      </div>
                    </div>
                  {/if}
                </div>
              </div>
            </div>

            <div class="accordion-item">
              <h2 class="accordion-header">
                <button
                  class="accordion-button collapsed"
                  type="button"
                  data-bs-toggle="collapse"
                  data-bs-target="#collapseHumidity"
                  aria-expanded="true"
                  aria-controls="collapseHumidity"
                >
                  <div class="row">
                    <div class="col-1">
                      <i class="fa-solid fa-droplet text-info" />
                    </div>
                    <div class="col">Relative Humidity</div>
                  </div>
                </button>
              </h2>
              <div
                id="collapseHumidity"
                class="accordion-collapse collapse"
                data-bs-parent="#growProfileSettings"
              >
                <div class="accordion-body">
                  {#if !growProfile.lighting}
                    <div class="card">
                      <div class="card-body">
                        <div class="mb-2">
                          <input
                            type="range"
                            class="form-range"
                            min="0"
                            max="100"
                            step="5"
                            id="profileHumidityNight"
                            bind:value={growProfile.relative_humidity_night}
                            on:change={() => calculateVPD("night")}
                            aria-describedby="profileHumidityNightHelp"
                          />
                          <div
                            class="row text-center"
                            id="profileHumidityNightHelp"
                          >
                            <div class="col">
                              <div class="form-text mt-0">
                                Relative Humidity {growProfile.relative_humidity_night}%
                              </div>
                            </div>
                          </div>
                          <input
                            type="range"
                            class="form-range"
                            min="0"
                            max="10"
                            step="1"
                            id="profileHumidityNightAllowedDeviation"
                            bind:value={growProfile.relative_humidity_night_allowed_deviation}
                            aria-describedby="profileHumidityNightAllowedDeviationHelp"
                          />
                          <div
                            class="row text-center"
                            id="profileHumidityNightAllowedDeviationHelp"
                          >
                            <div class="col">
                              <div class="form-text mt-0">
                                Relative Humidity allowed deviation ± {growProfile.relative_humidity_night_allowed_deviation}%
                              </div>
                            </div>
                          </div>
                        </div>
                      </div>
                    </div>
                  {:else if growProfile.lights_off == 1440}
                    <div class="card mb-2">
                      <div class="card-body">
                        <div class="mb-2">
                          <input
                            type="range"
                            class="form-range"
                            min="0"
                            max="100"
                            step="5"
                            id="profileHumidityDay"
                            bind:value={growProfile.relative_humidity_day}
                            on:change={() => calculateVPD("day")}
                            aria-describedby="profileHumidityDayHelp"
                          />
                          <div
                            class="row text-center"
                            id="profileHumidityDayHelp"
                          >
                            <div class="col">
                              <div class="form-text mt-0">
                                Relative Humidity {growProfile.relative_humidity_day}%
                              </div>
                            </div>
                          </div>
                          <input
                            type="range"
                            class="form-range"
                            min="0"
                            max="10"
                            step="1"
                            id="profileHumidityDayAllowedDeviation"
                            bind:value={growProfile.relative_humidity_day_allowed_deviation}
                            aria-describedby="profileHumidityDayAllowedDeviationHelp"
                          />
                          <div
                            class="row text-center"
                            id="profileHumidityDayAllowedDeviationHelp"
                          >
                            <div class="col">
                              <div class="form-text mt-0">
                                Relative Humidity allowed deviation ± {growProfile.relative_humidity_day_allowed_deviation}%
                              </div>
                            </div>
                          </div>
                        </div>
                      </div>
                    </div>
                  {:else}
                    <div class="card mb-2">
                      <div class="card-body">
                        <h6 class="card-title">
                          <i class="fa-regular fa-sun text-warning me-2" />Day
                        </h6>
                        <div class="mb-2">
                          <input
                            type="range"
                            class="form-range"
                            min="0"
                            max="100"
                            step="5"
                            id="profileHumidityDay"
                            bind:value={growProfile.relative_humidity_day}
                            on:change={() => calculateVPD("day")}
                            aria-describedby="profileHumidityDayHelp"
                          />
                          <div
                            class="row text-center"
                            id="profileHumidityDayHelp"
                          >
                            <div class="col">
                              <div class="form-text mt-0">
                                Relative Humidity {growProfile.relative_humidity_day}%
                              </div>
                            </div>
                          </div>
                          <input
                            type="range"
                            class="form-range"
                            min="0"
                            max="10"
                            step="1"
                            id="profileHumidityDayAllowedDeviation"
                            bind:value={growProfile.relative_humidity_day_allowed_deviation}
                            aria-describedby="profileHumidityDayAllowedDeviationHelp"
                          />
                          <div
                            class="row text-center"
                            id="profileHumidityDayAllowedDeviationHelp"
                          >
                            <div class="col">
                              <div class="form-text mt-0">
                                Relative Humidity allowed deviation ± {growProfile.relative_humidity_day_allowed_deviation}%
                              </div>
                            </div>
                          </div>
                        </div>
                      </div>
                    </div>
                    <div class="card">
                      <div class="card-body">
                        <h6 class="card-title">
                          <i class="fa-regular fa-moon me-2" />Night
                        </h6>
                        <div class="mb-2">
                          <input
                            type="range"
                            class="form-range"
                            min="0"
                            max="100"
                            step="5"
                            id="profileHumidityNight"
                            bind:value={growProfile.relative_humidity_night}
                            on:change={() => calculateVPD("night")}
                            aria-describedby="profileHumidityNightHelp"
                          />
                          <div
                            class="row text-center"
                            id="profileHumidityNightHelp"
                          >
                            <div class="col">
                              <div class="form-text mt-0">
                                Relative Humidity {growProfile.relative_humidity_night}%
                              </div>
                            </div>
                          </div>
                          <input
                            type="range"
                            class="form-range"
                            min="0"
                            max="10"
                            step="1"
                            id="profileHumidityNightAllowedDeviation"
                            bind:value={growProfile.relative_humidity_night_allowed_deviation}
                            aria-describedby="profileHumidityNightAllowedDeviationHelp"
                          />
                          <div
                            class="row text-center"
                            id="profileHumidityNightAllowedDeviationHelp"
                          >
                            <div class="col">
                              <div class="form-text mt-0">
                                Relative Humidity allowed deviation ± {growProfile.relative_humidity_night_allowed_deviation}%
                              </div>
                            </div>
                          </div>
                        </div>
                      </div>
                    </div>
                  {/if}
                </div>
              </div>
            </div>
          </div>
        </form>
      </div>
    </div>
  </div>
</div>

<style>
  .no-accordion-icon:after {
    background-image: none !important;
  }
</style>
