# AAP workflows + artefacts + in memory inventories

Job templates and workflow templates in Ansible Automation Platform can take extra variables as input.

This is useful to provide a workflow with information such as the size of the VM, the type of app that should be deployed.

A little known feature is that a step in the workflow can augment the original variables with new information.

Imagine a third party system that takes care of generating the hostname of the new VM.

We can actually reuse this new info within the workflow using the `ansible.builtin.set_stats` module.

In the following fictive example, a workflow is made of 3 steps:

- step 1: take the original input (variables), generate a random hostname, saving the extra info (called artefacts) using the set_stats module (once set_stats has been called, the vars are going to live for the duration of the workflow, you don't have to use set_stats at every steps of the workflow).
- step 2: the vars and artefacts from the previous steps are now the input for step2. This step just displays all the vars
- step 3: store the new hostname create under step 1 in an "in memory inventory". We can run some automation against this temporary inventory

When the machine has been provisioned successfully, we can update our CMDB.

It is obviously possible to update the CMDB at the end of step 1, refresh the inventory in step 2 and run automation with a limit in step 3, but it is overall slower and doesn't necessarily provide added value.


If you want a live demo, reach out to swains@redhat.com.
