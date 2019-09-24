## Purpose

The TMS Routing Service is responsible to route incoming messages to the
appropriate workflow and finally handle the request message. The workflow
is defined as a BMPN conform workflow, with [Activiti](https://www.activiti.org)
extensions, and comprises the execution of services in sequence order or in parallel
execution steps.

A typical and simple workflow to handle incoming [OSIP][2]
REQ_ messages could look like this:

![Workflow][1]
 
(1) Whenever a REQ_ message arrives, the `TransportUnit` with the given
`Barcode` is booked onto the `Location` provided in the message.

(2) For the current `TransportUnit` a `TransportOrder` is created with
the target `LocationGroup` equals to `FGSTOCK`.

(3) In the final step the system triggers a service to send an [OSIP][2] RES_
message to the underlying subsystem (i.e. PLC or Raspberry Pi) with the
next `Location` coordinate to move the `TransportUnit` to.

The actual [Activiti](https://www.activiti.org) BPMN snippet looks like this:

```
    <serviceTask id="sid-...27D" name="Book TransportUnit to current Location" activiti:expression="#{transportUnitApi.updateActualLocation(in.msg.barcode, in.msg.actualLocation)}"></serviceTask>
    <serviceTask id="sid-...B7F" name="Create TransportOrder to Stock" activiti:expression="#{transportOrderApi.createTO(&quot;FGSTOCK&quot;)}"></serviceTask>
    <serviceTask id="sid-...C72" name="Send RES_ to Conveyor location" activiti:expression="#{responder.sendTo(&quot;FGIN/CONV/0001/0000/0000&quot;)}"></serviceTask>
```

Notice that the flow and the expressions are modelled in [Activiti](https://www.activiti.org)
Model Explorer rather than in XML.

## Resources

[![Build status](https://travis-ci.com/openwms/org.openwms.tms.routing.svg?branch=master)](https://travis-ci.com/openwms/org.openwms.tms.routing)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Quality](https://sonarcloud.io/api/project_badges/measure?project=org.openwms:org.openwms.tms.routing&metric=alert_status)](https://sonarcloud.io/dashboard?id=org.openwms:org.openwms.tms.routing)
[![Join the chat at https://gitter.im/openwms/org.openwms](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/openwms/org.openwms?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

## Deployment
This component is a crucial part of the TMS (Transport Management System) and is required
in all automated warehouse projects. The microservice is deployed as RTU
(ready-to-use) box in following environments:

| endpoints | billed | SLA |
| --------- | ------ | --- |
| http://tms-routing-infrastructure.app.gcaas.ch | no | No SLA available, EU-CH region |

## Benefits

Why did we choose BPMN and [Activiti](https://www.activiti.org)? From the project experience we have,
we think that it is important that project consultants as well as
business people should realize the project specific parts in BPMN and not
in code. Usually after doing several projects an engineer has a library
of business functions and can just plug them together in some kind of
flow - we've misused the concept of BPMN for that.

## Features

The current implementation allows to reference standard Spring managed
beans within BPMN workflows. Remote http accessible microservices can
be accessed with support of `Feign`. Therefor the TMS Routing service
needs to have the Feign client interface definitions on the classpath at
startup.

[1]: images/workflow.png
[2]: https://www.interface21.io/osip
