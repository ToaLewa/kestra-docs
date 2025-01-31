---
title: Scheduler
icon: /docs/icons/architecture.svg
---

Scheduler is a server component responsible for processing all triggers except for the Flow Triggers (managed by the executor).

It continuously watches all the triggers and, if all conditions are met, will start an execution of the flow (submit the flow to the Executor).

In the case of polling triggers, the Scheduler will decide (based on the configured evaluation interval) whether to execute the flow. If the polling trigger conditions are met, the Scheduler will send the execution, along with the trigger metadata, to the Worker for execution.

Note that polling triggers cannot be evaluated concurrently. They also can't be reevaluated i.e. if the execution of a flow, which started as a result of the same trigger, is already in a Running state, the next execution will not be started until the previous one finishes.

Internally, the Scheduler will keep checking every second whether some trigger must be evaluated.

::alert{type="warning"}
By default, Kestra will handle all dates based on your system timezone. You can change the timezone with [JVM options](../10.administrator-guide/01.configuration/04.others.md#jvm-configuration).
::