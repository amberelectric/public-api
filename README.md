# Amber Public API forums

Discuss the [Amber public API](https://github.com/amberelectric/public-api/discussions)

## Home Assistant Blueprints

### Amber Light

Change the colour of a lamp based on the current Amber price!

This blueprint looks at the current price you are paying for electricity, and changes the colour of an RGB lamp, so you can decided whether to put the washing machine or dryer on.

The colors:

    Cyan: Extremely low price
    Green: Low price
    Orange: Average price
    Red: High price
    Purple: Price spike

You can change the thresholds when configuring the automation.

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Famberelectric%2Fpublic-api%2Fblob%2Fmain%2Fhome-assistant%2Fblueprints%2Flight.yaml)

### Best Price Block

This script will tell you the best time to use your appliances. You can ask for either the cheapest or most expensive block of time, and it will tell you in how many hours that block will, and what the average price would be.

Fields:

```
entity_id: "The Amber price forecast sensor to use. If in doubt, use the general forecast."
block_size: "The size of the block in hours."
min_max: "Min: cheapest Max: most expensive block"
```

You can call the blueprint as a service, and it will return the following object:

```
block_size: number,
starts_in_hours: number,
cents_per_kwh: number
min_max: min | max
```

![Call the script as a service](https://github.com/amberelectric/public-api/blob/main/home-assistant/blueprints/assets/best_price_example.png?raw=true)

#### Example conversation script

```yaml
alias: "Sentence: Next cheapest power block"
description: Ask when the next cheapest block of time is.
trigger:
  - platform: conversation
    command:
      - "[What is |When is ]the [next] cheapest {hour} hour block"
condition: []
action:
  - service: script.best_power_price
    data:
      min_max: min
      entity_id: sensor.amber_general_forecast
      block_size: "{{ trigger.slots.hour }}"
    response_variable: response
  - set_conversation_response: |-
      {% if response.starts_in_hours == 0 -%}
        The cheapest {{ response.block_size }} hour block is now, with an average price of {{ response.cents_per_kwh }} cents per kilowatt hour.
      {% elif response.starts_in_hours == 0.5 %}
        The cheapest {{ response.block_size }} hour block starts in 30 minutes, with an average price of {{ response.cents_per_kwh }} cents per kilowatt hour.
      {% else %}
        The cheapest {{ response.block_size }} hour block starts in {{response.starts_in_hours}} hours, with an average price of {{ response.cents_per_kwh }} cents per kilowatt hour.
      {% endif %}
mode: single
```

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Famberelectric%2Fpublic-api%2Fblob%2Fmain%2Fhome-assistant%2Fblueprints%2Fbest_block.yaml)
