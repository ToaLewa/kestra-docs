---
title: Schedule trigger
icon: /docs/icons/components.svg
---

The Schedule trigger generates new executions on a regular cadence based on a Cron expression or custom scheduling conditions.

```yaml
type: "io.kestra.core.models.triggers.types.Schedule"
```

> Kestra is able to trigger flows based on a Schedule (aka the time). If you need to wait for another system to be ready and cannot use any event mechanism, you can schedule one or more time the current flow.

## Example

> A schedule that runs every quarter of an hour.

```yaml
triggers:
  - id: schedule
    type: io.kestra.core.models.triggers.types.Schedule
    cron: "*/15 * * * *"
```

> A schedule that runs only the first monday of every month at 11 AM.
>
```yaml
triggers:
  - id: schedule
    type: io.kestra.core.models.triggers.types.Schedule
    cron: "0 11 * * 1"
    conditions:
      - type: io.kestra.core.models.conditions.types.DayWeekInMonthCondition
        date: "{{ trigger.date }}"
        dayOfWeek: "MONDAY"
        dayInMonth: "FIRST"
```

::alert{type="warning"}
Schedules **cannot overlap**. This means that we **cannot have any concurrent schedules**. If the previous schedule is not ended when the next one must start, the scheduler will wait until the end of the previous one. The same applies during backfills.
::

::alert{type="info"}
Most of the time, schedule execution will depend on the `trigger.date` (looking at files for today, SQL query with the schedule date in the where clause, ...). This works well but prevents you from executing your flow manually (since these variables are only available during the schedule).

You can use this expression to make your **manual execution work**: `{{ trigger.date ?? execution.startDate | date("yyyy-MM-dd") }}`. It will use the current date if there is no schedule date making it possible to start the flow manually.
::


## Backfill

Kestra will optionally handle schedule [backfills](../../07.concepts/backfill.md).


## Trigger expressions

When the flow is scheduled, some context expressions are injected to allow flow customization (such as filename, where clause in SQL query, etc.).

| Parameter                | Description                        |
|--------------------------|------------------------------------|
| `{{ trigger.date }}`     | the date of the current schedule.  |
| `{{ trigger.next }}`     | the date of the next schedule.     |
| `{{ trigger.previous }}` | the date of the previous schedule. |


## Schedule Conditions

When the `cron` is not sufficient to determine the date you want to schedule your flow, you can use `conditions` to add some additional conditions, (for example, only the first day of the month, only the weekend, ...).
You **must** use the `{{ trigger.date }}` expression on the property `date` of the current schedule.
This condition will be evaluated and `{{ trigger.previous }}` and `{{ trigger.next }}` will reflect the date **with** the conditions applied.

The list of core conditions that can be used are:

 - [DateTimeBetweenCondition](/plugins/core/conditions/io.kestra.core.models.conditions.types.datetimebetweencondition)
 - [DayWeekCondition](/plugins/core/conditions/io.kestra.core.models.conditions.types.dayweekcondition)
 - [DayWeekInMonthCondition](/plugins/core/conditions/io.kestra.core.models.conditions.types.dayweekinmonthcondition)
 - [NotCondition](/plugins/core/conditions/io.kestra.core.models.conditions.types.notcondition)
 - [OrCondition](/plugins/core/conditions/io.kestra.core.models.conditions.types.orcondition)
 - [WeekendCondition](/plugins/core/conditions/io.kestra.core.models.conditions.types.weekendcondition)
 - [DayWeekInMonthCondition](/plugins/core/conditions/io.kestra.core.models.conditions.types.dayweekinmonthcondition)

## Properties and Outputs

Check the [Schedule task](/plugins/core/triggers/io.kestra.core.models.triggers.types.schedule) documentation for the list of the task properties and outputs.

## Recover Missed Schedules

If a schedule is missed, Kestra will automatically recover it by default. This means that if the Kestra server is down, the missed schedules will be executed as soon as the server is back up. However, this behavior is not always desirable, e.g. during a planned maintenance window. In Kestra 0.15 and higher, this behavior can be disabled by setting the `recoverMissedSchedules` configuration to `NONE`.

Kestra 0.15 introduced a new configuration allowing you to choose whether you want to recover missed schedules or not:

```yaml
kestra:
  plugins:
    configurations:
      - type: io.kestra.core.models.triggers.types.Schedule
        values:
          # available options: LAST | NONE | ALL -- default: ALL
          recoverMissedSchedules: NONE
```

The `recoverMissedSchedules` configuration can be set to `ALL`, `NONE` or `LAST`:
- `ALL`: Kestra will recover all missed schedules. This is the **default** value.
- `NONE`: Kestra will not recover any missed schedules.
- `LAST`: Kestra will recover only the last missed schedule for each flow.

Note that this is a global configuration that will apply to all flows, unless other behavior is explicitly defined within the flow definition:

```yaml
triggers:
  - id: schedule
    type: io.kestra.core.models.triggers.types.Schedule
    cron: "*/15 * * * *"
    recoverMissedSchedules: NONE
```

In this example, the `recoverMissedSchedules` is set to `NONE`, which means that Kestra will not recover any missed schedules for this specific flow regardless of the global configuration.
