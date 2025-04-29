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

The workflow has 3 steps / job templates chained together:

### Step 1

Takes the original input (extra vars), generate a random hostname, saving this extra info (called "artefacts") using the `set_stats` module. Once `set_stats` has been called, the vars are going to live for the duration of the workflow, you don't have to use set_stats at every steps of the workflow.

### Step 2

The vars and artefacts from the previous steps are now the extra vars for step 2. 

This step just displays all the vars

### Step 3

We still have all the info (original + new data) as input.

We store the new hostname created under step 1 in an "in memory inventory". 

We can now run some automation against this temporary inventory

### Next steps?

When the machine has been provisioned successfully, we can update our CMDB.

## Why not using a dynamic inventory?

It is obviously possible to update the CMDB at the end of step 1, refresh the inventory in step 2 and run automation with a limit in step 3, but it is overall slower and doesn't necessarily provide any added value.


If you want a live demo, reach out to swains@redhat.com.
