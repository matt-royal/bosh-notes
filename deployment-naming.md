# Naming

Currently multiple things are named jobs which is confusing in code or in operation. Below diagram shows new proposed naming hierarchy:

```
Director (prod)
 |
 +-- Deployment (cf1)
      |
      +-- Group (cell-linux) *new*
           |
           +-- Member (2f66d0c7-b182-4f3e-96dd-a09ebaef2ae4) *new*
                |
                +-- Job (cell)
                     |
                     +-- Process (cell)
```

Director: Main orchestration components which is accessed by multiple users.

Deployment: Logically wraps together allocated cloud resources, saved configurations, used release versions, ACLs, etc. Typically user operates on a deployment which ties multiple releases and stemcells.

Group: Collection of X number of Members tasked to do same thing.

Member: Part of a pool doing a specific thing. Each instance in a cluster is configured in a similar way in regards to machine size, disk size, OS, configuration values, etc.

Job: Represents a specific thing to do on an Member. Could be placed with other jobs on a instance to cooperate. Typically is long running.

Process: Actual implementation of a Job. One or more processes may be needed to perform a Job. Processes are monitored and restarted.

## Alternatives

- cluster and node
- cluster and machine
- group and member
  - con: not necessarily specific? groups of what?
- service and instance
  - con: job groupings do not represent logical groups but rather machine locations

## Example Manifest

```yaml
releases:
- name: loggregator
  version: latest
- name: web
  version: latest
- name: postgres
  version: latest

stemcells:
- alias: default
  os: ubuntu-trusty
  version: latest

clusters:
- name: db
  nodes: 1
  stemcell: default
  vm_type: default
  persistent_disk_type: small
  jobs:
  - {name: postgres, release: postgres}
  - {name: metron_agent, release: loggregator}
- name: web
  nodes: 1
  stemcell: default
  vm_type: default
  jobs:
  - {name: web, release: web}
  - {name: metron_agent, release: loggregator}
```

## Stories

- rename deployment jobs to cluster
  - must be backwards compatible so allow `jobs:` in the manifest
  - raise an error if both jobs and clusters keys are specified
- rename templates to jobs
  - must be backwards compatible so allow `templates:` in the manifest
  - raise an error if both templates and jobs keys are specified
