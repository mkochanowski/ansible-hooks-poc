# Ansible Hooks PoC

This repository contains a basic PoC for an extensions mechanism for Ansible using hooks.  

## Directory structure

`hooks/` --  contains hooks that can be injected during playbook or role execution. Hooks can be tracked in a separate repository/VCS.

`project/` -- contains a sample Ansible project. Run the `test-hooks.yml` playbook to test how hook injections work.

## Sample execution

```shell
$ ansible-playbook test-hooks.yml -i inventory.yml 

PLAY [Test hooks]

TASK [test_hooks : debug]
ok: [localhost] => {
    "msg": "Task 1"
}

TASK [test_hooks : debug]
ok: [localhost] => {
    "msg": "Task 2"
}

TASK [include_role : inject_hook] 

TASK [inject_hook : debug] 
ok: [localhost] => {
    "msg": "Injecting 'print_custom_debug_message' for 'test_hooks' if such hook exists..."
}

TASK [inject_hook : include_tasks] 
included: ${PROJECT_DIR}/hooks/test_hooks/print_custom_debug_message.yml for localhost

TASK [inject_hook : debug] 
ok: [localhost] => {
    "msg": "This is a debug message called from the injected hook."
}

TASK [test_hooks : debug]
ok: [localhost] => {
    "msg": "Task 3"
}

TASK [test_hooks : debug]
ok: [localhost] => {
    "msg": "Task 4"
}

TASK [test_hooks : debug]
ok: [localhost] => {
    "custom_fact_set_in_hook": "VARIABLE IS NOT DEFINED!"
}

TASK [include_role : inject_hook]

TASK [inject_hook : debug]
ok: [localhost] => {
    "msg": "Injecting 'set_custom_facts' for 'test_hooks' if such hook exists..."
}

TASK [inject_hook : include_tasks] 
included: ${PROJECT_DIR}/hooks/test_hooks/set_custom_facts.yml for localhost

TASK [inject_hook : set_fact]
ok: [localhost]

TASK [test_hooks : debug]
ok: [localhost] => {
    "custom_fact_set_in_hook": "This value was set in a hook"
}

PLAY RECAP ***********************
localhost                  : ok=12   changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
