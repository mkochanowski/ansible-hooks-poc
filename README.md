# Ansible Hooks PoC

This repository contains a PoC for an extension mechanism for Ansible using injection hooks that can be referenced and executed during an Ansible Play.  

## Directory structure

- `hooks/` --  contains hooks that can be injected during playbook or role execution. Ideally, this would be tracked in a separate repository in VCS.
- `project/` -- contains a sample Ansible project. Contains the `test-hooks.yml` playbook that starts tests included in this repository.
- `project/roles/inject_hook/` -- the essential piece of this PoC. This role contains logic for injecting hooks during an Ansible Play.
- `project/roles/test_hooks/` -- helper role that houses different tests for injection hooks. To see a list of included tests, see the section below.

## Included tests

The following "tests" are included in the sample project:

- `test_hook_missing.yml` -- tests whether the Play will continue if a hook injection request references a hook that does not exist.
- `test_set_custom_facts.yml` -- tests whether facts set by the injected hook persist during the Play.
- `test_simple_debug_injection.yml` -- tests basic functionality of embedding custom logic into the running Play. In this case, this is done by printing a debug statement with a custom message.

## Running the provided example

To run all tests, go to the `project/` directory and issue the following command:

```shell
$ ansible-playbook test-hooks.yml -i inventory.yml 
```

This command will run all tests defined in the `test_hooks` role of the project.

## Sample output of the execution

```shell
$ ansible-playbook test-hooks.yml -i inventory.yml 

PLAY [Test hooks] 

TASK [test_hooks : Include tasks containing tests] 
included: project/roles/test_hooks/tasks/test_hook_missing.yml for localhost
included: project/roles/test_hooks/tasks/test_set_custom_facts.yml for localhost
included: project/roles/test_hooks/tasks/test_simple_debug_injection.yml for localhost

TASK [test_hooks : debug]
ok: [localhost] => {
    "msg": "Test: Ensure missing hook does not terminate playbook execution"
}

TASK [include_role : inject_hook]

TASK [inject_hook : debug] 
ok: [localhost] => {
    "msg": "Injecting 'test_hooks/this_hook_should_not_exist' if such hook exists..."
}

TASK [inject_hook : include_tasks]
skipping: [localhost]

TASK [test_hooks : debug]
ok: [localhost] => {
    "msg": "Test: Set Custom Facts from within the Injected Hook"
}

TASK [test_hooks : debug]
ok: [localhost] => {
    "custom_fact_set_in_hook": "VARIABLE IS NOT DEFINED!"
}

TASK [include_role : inject_hook]

TASK [inject_hook : debug] 
ok: [localhost] => {
    "msg": "Injecting 'test_hooks/set_custom_facts' if such hook exists..."
}

TASK [inject_hook : include_tasks]
included: /home/mkochano/Projects/current/ansible-hooks-poc/hooks/test_hooks/set_custom_facts.yml for localhost

TASK [inject_hook : set_fact] 
ok: [localhost]

TASK [test_hooks : debug]
ok: [localhost] => {
    "custom_fact_set_in_hook": "This value was set in a hook! Success!"
}

TASK [test_hooks : debug]
ok: [localhost] => {
    "msg": "Test: Inject simple debug statement"
}

TASK [include_role : inject_hook]

TASK [inject_hook : debug] 
ok: [localhost] => {
    "msg": "Injecting 'test_hooks/print_debug_message' if such hook exists..."
}

TASK [inject_hook : include_tasks]
included: /home/mkochano/Projects/current/ansible-hooks-poc/hooks/test_hooks/print_debug_message.yml for localhost

TASK [inject_hook : debug] 
ok: [localhost] => {
    "msg": "This is a debug message called from the injected hook."
}

PLAY RECAP
localhost                  : ok=15   changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   


```
