---
layout: wmt/docs
title:  Node Roster
side-navigation: wmt/docs-navigation.html
---

# {{ page.title }}

**Note:** Node Roster is a "beta" feature. The plugin's actions and parameters
are subject to change.

The Node Roster task provides a way to access [Node Roster](../getting-started/node-roster.html)
data in Concord flows.

- [Usage](#usage)
    - [Common Parameters](#common-parameters)
    - [Result Format](#result-format)
    - [Find the Last Deployer](#find-the-last-deployer)
    - [Find Hosts by Project](#find-hosts-by-project)
    - [Find Hosts by Artifact](#find-hosts-by-artifact)
    - [Get All Known Hosts](#get-all-known-hosts)
    - [Get Host Facts](#get-host-facts)
    - [Find Artifacts By Host](#find-artifacts-by-host)

## Usage

To be able to use the task in a Concord flow, it must be added as a
[dependency](../getting-started/concord-dsl.html#dependencies):

```yaml
configuration:
  dependencies:
  - mvn://com.walmartlabs.concord.plugins.basic:noderoster-tasks:{{ site.concord_core_version }}
```

This adds the task to the classpath and allows you to invoke the task in
a flow:

```yaml
flows:
  default:
    - task: nodeRoster
      in:
        action: "deployedBy"
        hostName: "myhost.example.com"
```

### Common Parameters

- `baseUrl` - (optional) string, base URL of the Concord API. If not set uses the
  current instance's API address.
- `action` - string, name of the action.

### Result Format

The task's actions return their results in a `result` variable. The variable
has the following format:

- `ok` - boolean, `true` if the operation was successful - i.e. returned some
  data;
- `data` - object, result of the operation.

### Find Hosts by Artifact

Returns a list of hosts which had the specified artifact deployed to.

```yaml
- task: nodeRoster
  in:
    action: "hostsWithArtifacts"
    artifactPattern: ".*my-app-1.0.0.jar"
```

Parameters:
- `artifactPattern` - regex, name or pattern of the artifact's URL;
- `limit` - number, maximum number of records to return. Default is `30`;
- `offset` - number, offset of the first record, used for paging. Default
  is `0`.

The action returns the following `result`:

```json
{
  "ok": true,
  "data": {
    "artifact A": [
      { "hostId":  "...", "hostName": "..."},
      { "hostId":  "...", "hostName": "..."},
      ...
    ],
    "artifact B": [
      { "hostId":  "...", "hostName": "..."},
      { "hostId":  "...", "hostName": "..."},
      ...
    ]
  }
}
```

The `data` is a object where keys are artifact URLs matching the supplied
`artifactPattern` and values are lists of hosts. 

### Get Host Facts

Returns the last registered snapshot of the host's [Ansible facts](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#variables-discovered-from-systems-facts).

```yaml
- task: nodeRoster
  in:
    action: "facts"
```

Parameters:
- `hostName` - string, name of the host to look up;
- `hostId` - UUID, id of the host to look up.

The action returns the following `result`:

```json
{
  "ok": true,
  "data": {
    ...
  }
}
```

The `data` value is the fact's JSON object as it was received from Ansible.

### Find Artifacts By Host

Returns a list of artifacts deployed on the specified host.

```yaml
- task: nodeRoster
  in:
    action: "deployedOn"
    hostName: "myhost.example.com"
```

Parameters:
- `hostName` - string, name of the host to look up;
- `hostId` - UUID, id of the host to look up.

Either `hostName` or `hostId` are required.

The action returns the following `result`:

```json
{
  "ok": true,
  "data": [
    { "url": "..." },
    { "url": "..." }
  ]
}
```
