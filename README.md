# AAP workflows + artefacts + in memory inventories

Note: artifact and artefact are both valid in the English language. I might use both versions artifact (US) and artefact (UK) across this page.

Job templates and workflow templates in Ansible Automation Platform (AAP) accept extra variables as input / parameters.

Parameters can be things like the size of the VM to be deployed, the type of middleware that should be installed, the DNS resolver to use, etc.

A little known feature is that AAP workflows can augment the original extra variables with new data / info.

Imagine a third party system that takes care of generating the hostname of a new VM.

We can capture this new info using `ansible.builtin.set_stats` module and it will live for the duration of the workflow, for other steps to "consume".

## Scenario

In the following fictive example, a workflow takes some parameters as input (extra vars):

```
payload_original:
  requested_os: linux
  vm_size: small
  owner: Matt Hicks
  app: jboss
```

The scope of those extra vars is the workflow. Those vars will be available for each steps in the workflow (unless they are overriden or unset).

The following workflow has 3 steps ("job templates") chained together:

### Step 1

This step receives the original extra vars passed to the workflow, untouched at this point.

Then, I generate a random hostname and I save this extra info (called "artifacts" in the workflow output) using the `set_stats` module.

![](https://raw.githubusercontent.com/sebw/AAP2-workflow-artefact/refs/heads/master/step1.png)

### Step 2

The original extra vars (passed to the workflow) and artefacts (created at step 1) are now the extra vars for step 2!

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

The extra vars are similar to step 2, as we didn't change anything in the previous step.

We use the `generated_hostname` variable in a "in memory inventory" so we can immediately act on that hostname.

Here, I'm trying to install a package against this in memory inventory.

Note: the in memory inventory is only valid during the job template execution. You need to recreate the inventory at each step, if needed.

### Next steps?

When our fictive VM has been provisioned successfully, we can finally update the CMDB / real inventory.

## Why not using a dynamic inventory in step 2?

It is obviously possible to update the CMDB and dynamic inventory once the hostname has been generated.

If you do that, you need to:

- update the inventory (maybe with incomplete info at this point)
- refresh the inventory
- run the next job template against the dynamic inventory and use the "limit" to target the new VM

To me, at this point, this is often:

- time consuming: a dynamic inventory update and sync can take from seconds to minutes depending on the size of the inventory
- risky: if you fail to set the limit properly, you might run a job template against incorrect machines or even the full inventory

From a CMDB standpoint, I prefer updating the CMDB/inventory only when it makes sense.  
I need the guarantee the new resource is compliant to the corporate standards before I add it to the CMDB.

A workflow can fail in many ways at every steps and I want to avoid potentially bogus or incomplete data in the CMDB.

There are obviously scenarios where updating the inventory makes sense. For example if you do "workflows in workflows" and the second workflow is used in many scenarios and designed to use dynamic inventories, regardless of how it's being consumed.

## Want a demo?

If you want a live demo, reach out to swains@redhat.com.
