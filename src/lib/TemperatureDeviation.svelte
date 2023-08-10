<script>
  export let celsius = 0;
  export let scale = "celsius";
  let display = calculate();

  $: celsius, scale, (display = calculate());

  function calculate() {
    switch (scale) {
      case "celsius":
        return celsius.toFixed(1) + "°C";
      case "fahrenheit":
        const f = celsius * (9 / 5);
        return f.toFixed(1) + "°F";
      case "kelvin":
        const k = celsius;
        return k.toFixed(1) + "K";
    }
  }
</script>

<span data-temperature-celsius={celsius}>{display}</span>
