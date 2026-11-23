---
name: flow-authoring
description: >
  Flow authoring: write or change one BPMN 2.0 product flow, a process that runs across use cases,
  actors and features, with each step tagged with its epic, domains and use cases.
  Use when /ie/bootstrap, /ie/flow or /ie/extract-brief needs a flow file written, a target flow included.
  Not for the state machine or sequence of one use case (that is the use case's flow.mermaid).
---

# Flow Authoring Skill

## Purpose

A product flow describes what the application does end to end, such as onboarding a client from first contact to first sign-in. It is one BPMN 2.0 file holding one process. Each step can name the roadmap epic, the domain contexts and the use cases behind it, so a reader can go from the picture to the specs.

A flow describes the application as it is today. A hub flow at `product/flows/<flow-id>.bpmn` changes only when a feature or fix that changes it is delivered. A proposed change travels with its brief as a draft at `features/briefs/<slug>/flows/<flow-id>.bpmn`.

A target flow at `product/epics/flows/<flow-id>.bpmn` describes a process as it will be after its epics are delivered. A business analyst draws it before any feature exists, with `/ie/flow <flow-id> --target`. It uses the same rules as a hub flow. It may share its id with a hub flow, and then shows the target version of that process. Delivery never changes or deletes it.

## Rules

1. **Write the process, never the diagram.** Write only the process semantics: events, tasks, gateways, sequence flows and lanes. Never write a `bpmndi:BPMNDiagram` element or any coordinate. After every write, run `ie flow layout <file>`. It replaces the coordinates with a fresh automatic layout and refuses a file that is not a flow, so fix the file and run it again when it does.
2. **One process per file.** The process `id` is `process-<flow-id>`, because an XML id may not start with a digit. Give the process a `name` a reader recognises.
3. **The flow id** matches `^[a-z0-9][a-z0-9-]*$` and is the file name without `.bpmn`.
4. **Tag steps with the `ie` namespace.** Declare `xmlns:ie="https://rp-flow.dev/schema/bpmn/ie/1.0"` on `bpmn:definitions`. Each task may carry three attributes:
   - `ie:epic`: space-separated epic ids from the `epics` list of the `product/roadmap.md` frontmatter. A step that two epics change names both, such as `ie:epic="E-01 E-02"`.
   - `ie:domains`: space-separated context names from `domain-model.yml`.
   - `ie:useCases`: space-separated use case ids from `catalog.yml`, such as `UC-012`.
5. **Never invent a reference.** Take every epic, context and use case from the files above. When a step has no use case yet, leave `ie:useCases` out. Do not make up an id. `ie doctor` warns about each reference it cannot find.
6. **Model what an actor does, in order.** Use a `bpmn:userTask` for a step a person does and a `bpmn:serviceTask` for a step the system does. Use an exclusive gateway for a decision and name each outgoing sequence flow with its condition. Use a lane per actor when more than one actor takes part. Name each task with a verb and an object, such as "Verify email address".
7. **Keep element ids stable.** When you change a flow, keep the id of every element you do not remove, and give a new element a descriptive id such as `task-verify-email`. Stable ids keep the diff small enough to review.
8. **Write one process, not a feature list.** A flow shows the order in which things happen. Leave out steps that no actor or system takes, and do not repeat the detail a use case already holds.

## Skeleton

```xml
<?xml version="1.0" encoding="UTF-8"?>
<bpmn:definitions xmlns:bpmn="http://www.omg.org/spec/BPMN/20100524/MODEL"
                  xmlns:ie="https://rp-flow.dev/schema/bpmn/ie/1.0"
                  id="definitions-client-onboarding" targetNamespace="https://rp-flow.dev/flows">
  <bpmn:process id="process-client-onboarding" name="Client onboarding" isExecutable="false">
    <bpmn:laneSet id="lanes">
      <bpmn:lane id="lane-client" name="Client">
        <bpmn:flowNodeRef>task-register</bpmn:flowNodeRef>
      </bpmn:lane>
      <bpmn:lane id="lane-platform" name="Platform">
        <bpmn:flowNodeRef>task-send-invite</bpmn:flowNodeRef>
      </bpmn:lane>
    </bpmn:laneSet>
    <bpmn:startEvent id="start" name="Client asks for access"/>
    <bpmn:userTask id="task-register" name="Register account"
                   ie:epic="E-01" ie:domains="identity" ie:useCases="UC-001"/>
    <bpmn:serviceTask id="task-send-invite" name="Send sign-in invite"
                      ie:epic="E-01 E-02" ie:domains="identity notification" ie:useCases="UC-002"/>
    <bpmn:endEvent id="end" name="Client signed in"/>
    <bpmn:sequenceFlow id="flow-1" sourceRef="start" targetRef="task-register"/>
    <bpmn:sequenceFlow id="flow-2" sourceRef="task-register" targetRef="task-send-invite"/>
    <bpmn:sequenceFlow id="flow-3" sourceRef="task-send-invite" targetRef="end"/>
  </bpmn:process>
</bpmn:definitions>
```

The automatic layout gives lanes and participants no coordinates, so a viewer draws the steps without their lanes. The lanes stay in the process.

## Procedure

1. Read `product/roadmap.md`, `domain-model.yml`, `catalog.yml` and the flows under `product/flows/` and `product/epics/flows/`, so the steps reuse the hub's names and do not duplicate another flow.
2. Ask the engineer for the process: its trigger, its end state, the actors, and the steps in order with their decisions. Ask one question at a time. Propose the steps from the catalog and the roadmap, and let the engineer correct them.
3. Present the steps as a numbered list, each with its actor and its epic, domains and use cases. Write the file only after the engineer confirms the list.
4. Write the file at the path the calling command gives, then run `ie flow layout <file>`.
5. Run `ie doctor` and fix each warning it reports for this file.
6. Show the engineer the diff of the file.
