<script lang="ts" context="module">
  export type GrowProfile = {
    name: string;
    lighting: boolean;
    lights_off: number;
    minimum_intake: number;
    maximum_intake: number;
    minimum_exhaust: number;
    maximum_exhaust: number;
    temperature_day: number;
    temperature_day_allowed_deviation: number;
    temperature_night: number;
    temperature_night_allowed_deviation: number;
    relative_humidity_day: number;
    relative_humidity_day_allowed_deviation: number;
    relative_humidity_night: number;
    relative_humidity_night_allowed_deviation: number;
    vpd_day: number;
    vpd_night: number;
  };
</script>
