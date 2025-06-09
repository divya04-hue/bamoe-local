# BAMOE (Business Automation Manager Open Edition)

## Overview

**BAMOE** stands for **Business Automation Manager Open Edition**, Red Hat's **open source** platform for automating business processes, decisions, and workflows.

It brings together:
- **Drools** – A powerful business rule engine.
- **jBPM** – A workflow and Business Process Management engine.
- **Kogito** – A cloud-native runtime for business automation.

BAMOE is the upstream community version of **Red Hat Process Automation Manager (RHPAM)**.

---

## What Can You Do with BAMOE?

- Create **BPMN** workflows for business processes
- Design **DMN** (Decision Model and Notation) decision tables
- Define and run **business rules** with Drools
- Build **cloud-native**, container-ready microservices
- Integrate with **event-driven architectures** (Kafka, Knative, etc.)
- Store everything in Git and deploy using GitOps pipelines

---

## What is BAMOE Canvas?

**BAMOE Canvas** is a **web-based, visual editor** for building business logic without needing to write Java code.

### Features:
- Drag-and-drop interface for:
  - BPMN process modeling
  - DMN decision modeling
  - Business rule creation
- Git-based project storage
- Containerized deployment support
- Integration with CI/CD workflows

It’s ideal for:
- Business Analysts who want to model workflows
- DevOps Engineers building GitOps pipelines with decision automation
- Developers creating intelligent microservices

---

## Architecture Components

| Component | Description |
|----------|-------------|
| **Kogito** | Runtime engine optimized for Kubernetes/OpenShift |
| **Drools** | Rule engine for business logic |
| **jBPM** | Engine for BPMN workflows |
| **DMN** | Standard for decision tables and logic |
| **BAMOE Canvas** | Visual modeling UI for BPMN, DMN, and Rules |

---

## DevOps Integration

BAMOE plays well with modern DevOps stacks:

- Container-native (Kubernetes/OpenShift support)
- GitOps-friendly (models and rules in Git repos)
- CI/CD integration using Maven, Jenkins, Tekton, ArgoCD, etc.
- REST and event-driven APIs (Kafka support)
- Scalable, microservices architecture

---

## Quick Start

1. **Install Java and Maven**
2. **Clone a sample Kogito/BAMOE project**
3. **Model your business logic in BAMOE Canvas**
4. **Build using Maven**
5. **Deploy as a container (Podman/Docker/OpenShift)**

---

## Resources

- [Official BAMOE Docs](https://kiegroup.github.io/bamoe-docs/)
- [Kogito Project](https://kogito.kie.org/)
- [Drools Project](https://www.drools.org/)
- [jBPM Project](https://www.jbpm.org/)

---

## Use Case Example

> *"When a customer's credit score is low AND their balance is overdue, flag their account and route to the 'collections' workflow."*

You can define this as:
- A **DMN decision**
- A **Drools rule**
- A **BPMN workflow**
And deploy it as a RESTful service into your production cluster.

---

## Maintainers

Community project maintained under the [KIE Group](https://github.com/kiegroup).

---

## License

Apache License 2.0




















Using Apache Maven to Setup the Project


<kogito.bom.group-id>com.ibm.bamoe</kogito.bom.group-id>
<kogito.bom.artifact-id>bamoe-bom</kogito.bom.artifact-id>
<kogito.bom.version>9.0.0.Final</kogito.bom.version>

<groupId>${kogito.bom.group-id}</groupId>
<artifactId>${kogito.bom.artifact-id}</artifactId>
<version>${kogito.bom.version}</version>

<dependency>
<groupId>com.ibm.bamoe</groupId>
<artifactId>bamoe-ilmt-compliance-quarkus-pamoe</artifactId>
</dependency>

DivyaLakshmis-MacBook-Pro:~ divyalakshmi$ scp -r root@9.30.140.218:quick-kogito ~/Documents/







oc new-app quay.io/bamoe/canvas:9.0.1 --name=$APP_NAME_BAMOE_CANVAS \
  -e KIE_SANDBOX_EXTENDED_SERVICES_URL=https://$(oc get route $APP_NAME_EXTENDED_SERVICES --output jsonpath={.spec.host}) \
  -e KIE_SANDBOX_GIT_CORS_PROXY_URL=https://$(oc get route $APP_NAME_GIT_CORS_PROXY --output jsonpath={.spec.host})