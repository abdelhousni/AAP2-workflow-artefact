# AAP workflows + artefacts + in memory inventories

Job templates and workflow templates in Ansible Automation Platform can take extra variables as input.

This is useful to provide a workflow with information such as the size of the VM, the type of app that should be deployed.

A little known feature is that a step in the workflow can augment the original variables with new information.

Imagine a third party system that takes care of generating the hostname of the new VM.

We can actually reuse this new info within the workflow using the `ansible.builtin.set_stats` module.

## Scenario

In the following fictive example, a workflow takes some parameters as input:

```
payload_original:
  requested_os: linux
  vm_size: small
  owner: Matt Hicks
  app: jboss
```

The scope of those vars is the workflow. Those vars will survive for each step of the workflow (unless they are overriden or unset).

The workflow has 3 steps (called "job templates") chained together:

### Step 1

This step receives the original vars passed to the workflow.

I generate a random hostname and I save this extra info (called "artifacts" in the workflow output) using the `set_stats` module.

![](https://raw.githubusercontent.com/sebw/AAP2-workflow-artefact/refs/heads/master/step1.png)

### Step 2

The vars (passed to the workflow) and artefacts (created at step 1) are now the extra vars for step 2. 

```
payload_extra:
  generated_hostname: iv5cgtcv9q.labo.ovh
payload_original:
  requested_os: linux
  vm_size: small
  owner: Matt Hicks
  app: jboss
```

![](https://raw.githubusercontent.com/sebw/AAP2-workflow-artefact/refs/heads/master/step2.png)

This step just displays all the vars for the host.

### Step 3

The vars are similar to step 2, we didn't change anything in the previous step.

We use `generated_hostname` in a "in memory inventory" so we can act on that hostname without necessarily syncing with a dynamic inventory in AAP. 

On the next step, we can target the in memory inventory to perform some tasks.

### Next steps

When the machine has been provisioned successfully, we can update our CMDB / real inventory.

## Why not using a dynamic inventory in step 2?

It is obviously possible to update the CMDB and dynamic inventory once the hostname has been generated.

If you do that, you need to:

- refresh the inventory
- run the next job template against that inventory with a limit on the newly create resource

To me, this is:

- time consuming: a dynamic inventory sync can take from seconds to minutes depending on the size of the inventory
- risky: if you fail to set the limit properly, you might run a job template against incorrect machines or even the full inventory

From a CMDB standpoint, I prefer updating the CMDB/inventory only when it makes sense. I want ot have the guarantee the new resource is compliant to the corporate standards before I add it to the CMDB.

A workflow can fail in many ways at every step and I want to avoid potentially bogus or incomplete data in the CMDB.

## Want a demo?

If you want a live demo, reach out to swains@redhat.com.
