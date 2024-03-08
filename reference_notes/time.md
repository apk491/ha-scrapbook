*WORK IN PROGRESS*

Current time reference:
- `{{ now() }}`
- `{{ now() | as_local }}`
- `{{ now() | as_timestamp }}`

This returns as a datetime, so for the purposes of this template we are using a static timestamp:
```
eg_now   = {% set eg_now = "2024-03-05 09:08:07.123456+00:00" %}{{ eg_now }}
dt       = {% set dt = eg_now | as_datetime %}{{ dt }}
ts       = {% set ts = eg_now | as_timestamp %}{{ ts }}
```
This will return the following values, which will be used instead of `{{ now() }}` in this template:
- `{{ eg_now }}` = 2024-03-05 09:08:07.123456+00:00, which mimics `{{ now() }}`
- `{{ dt }}` = 2024-03-05 09:08:07.123456+00:00, which mimics `{{ now() | as_datetime }}`
- `{{ ts }}` = 1709629687.123456, which mimics `{{ now() | as_timestamp }}`

Set variables for timescales (optional):
```
{% set spm =     60 | int %} # 1 Minute (60 Seconds)
{% set sph =   3600 | int %} # 1 Hour (3600 Seconds)
{% set spd =  86400 | int %} # 1 Day (86400 Seconds)
{% set spw = 604800 | int %} # 1 Week (604800 Seconds)
{% set mpd =   1440 | int %}} # 1 Day (1440 Minutes)
{% set mpw =  10080 | int %}} # 1 Week (10080 Minutes)
{% set hpw =    168 | int %}} # 1 Week (168 Hours)
```

# Calculating Time
Using `timedelta` we can calculate changes in time when used with a **datetime**.

`{{ <YOUR_DATETIME> + timedelta(days=1, hours=2, minutes=3, seconds=4) }}`

Or, alternatively, with a **timestamp**: `{{ <YOUR_TIMESTAMP> | as_datetime + timedelta(days=1, hours=2, minutes=3, seconds=4) }}`
```
* CURRENT TIME :  {{ dt }}
+ 1 YEAR       :  {{ dt + timedelta(days=365) }}
- 1 DAY  (24H) :  {{ dt - timedelta(days=1) }}
+ 3 DAYS (72H) :  {{ dt + timedelta(days=3) }}
- 3 HOURS      :  {{ dt - timedelta(hours=3) }}
+ 1 HR 26 MIN  :  {{ dt + timedelta(hours=1, minutes=26) }}
+ 1D 2H 3M 4S  :  {{ dt + timedelta(days=1, hours=2, minutes=3, seconds=4) }}
```
**Timestamps** can be calculated by adding or subtracting your values in seconds, which is generally easier for smaller values:
```
* TIMESTAMP    :  {{ ts }}
* CURRENT TIME :  {{ ts | as_datetime  }}
+ 1 YEAR       :  {{ ( ts + (spd * 365) ) | as_datetime }}
- 1 DAY  (24H) :  {{ ( ts - spd ) | as_datetime }}
+ 3 DAYS (72H) :  {{ ( ts + (spd * 3) ) | as_datetime }}
- 3 HOURS      :  {{ ( ts - (sph * 3) ) | as_datetime }}
+ 1 HR 26 MIN  :  {{ ( ts + sph + (spm * 26) ) | as_datetime }}
+ 1D 2H 3M 4S  :  {{ ( ts + spd + (sph * 2) + (spm * 3) + 4 ) | as_datetime }}
```

You can use the `replace` function with a **datetime** to alter the time in your template:

```
{% set start_of_today = dt.replace(hour=0, minute=0, second=0, microsecond=0) %}
```
This can be combined with `timedelta` to determine a date or time:
```
{% set start_of_tomorrow = start_of_today + timedelta(days=1) %}
```

Use `relative_time` with a **datetime** that is set *in the past*:
```
relative_time: {{ relative_time( start_of_today ) }} ago
```
This can't be used with future dates, but you can work around it:
```
{% set current_time = dt  %}
{% set future_time = as_local(dt) %}
{% set time_distance = future_time - current_time %}
relative_future_time: In {{ relative_time(current_time - time_distance) }}
```
# Using Time Templates with history_stats
You can utilise time templates with Home Assistant's native [**History Stats**](https://www.home-assistant.io/integrations/history_stats/) integration.

This sensor will return a time value of how long a lamp has been 'on' today:
```
sensor:
  - platform: history_stats
    name: Lamp ON today
    entity_id: light.my_lamp
    state: "on"
    type: time
    start: "{{ now().replace(hour=0, minute=0, second=0) }}"
    end: "{{ now() }}"
```
This is just one of the [**various examples**](https://www.home-assistant.io/integrations/history_stats/#examples) provided in the documentation.

#Formatting The Time
You can use `strftime` with a **datetime** or `timestamp_custom` with a **timestamp** to format the time into a string.

Both share the same directives, where you can use as many as you like, incoroprating other characters:
- `{{ dt.strftime('%Y-%m-%dT%H:%M:%S') }}` = 2024-03-05T09:08:07
- `{{ dt.strftime('THE TIME IS %-I %p') }}` = THE TIME IS 9 AM

# NOT FINISHED
More to add later. Here's a completely unrelated table:

| Month    | Savings |
| -------- | ------- |
| January  | $250    |
| February | $80     |
| March    | $420    |
