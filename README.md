ansible-common-roles
====================

Using ansible 1.4.4.

When using Ansible roles I ran into a problem where common roles were being skipped. This repo is a demonstration of the issue. There are 2 roles, _role1_ and _role2_, each of which depend on a 3rd role, _common_.

I have three playbooks, _role1.yml_, _role2.yml_ and _playbook.yml_. The first playbook selects a role to execute based upon a variable passed via the command line. The second 2 simply execute _role1_ and _role2_ respectively.

Here is the result of running the _role1_ playbook:-

    ⚡ ansible-playbook -i inventory role1.yml

    PLAY [Playbook to do role 1] **************************************************

    GATHERING FACTS ***************************************************************
    ok: [abdul]

    TASK: [common | A common task] ************************************************
    changed: [abdul]

    TASK: [role1 | A role1 task] **************************************************
    changed: [abdul]

    PLAY RECAP ********************************************************************
    abdul                      : ok=3    changed=2    unreachable=0    failed=0

And as expected, running _role2_ is similar:-

    PLAY [Playbook to do role2] ***************************************************

    GATHERING FACTS ***************************************************************
    ok: [abdul]

    TASK: [common | A common task] ************************************************
    changed: [abdul]

    TASK: [role2 | A role2 task] **************************************************
    changed: [abdul]

    PLAY RECAP ********************************************************************
    abdul                      : ok=3    changed=2    unreachable=0    failed=0

But see what happens when I run _playbook.yml_, wherein I select the role to run via a CL variable. First select to run _role1_:-

⚡ ansible-playbook -i inventory playbook.yml -e do_role=role1

    PLAY [Configure depending on passed in variable] ******************************

    GATHERING FACTS ***************************************************************
    ok: [abdul]

    TASK: [common | A common task] ************************************************
    changed: [abdul]

    TASK: [role1 | A role1 task] **************************************************
    changed: [abdul]

    TASK: [role2 | A role2 task] **************************************************
    skipping: [abdul]

    PLAY RECAP ********************************************************************
    abdul                      : ok=3    changed=2    unreachable=0    failed=0

The roles _common_ and _role1_ are run, and _role2_ is skipped. All ok.

Now I select _role2_ in _playbook.yml__:-

⚡ ansible-playbook -i inventory playbook.yml -e do_role=role2

    PLAY [Configure depending on passed in variable] ******************************

    GATHERING FACTS ***************************************************************
    ok: [abdul]

    TASK: [common | A common task] ************************************************
    skipping: [abdul]

    TASK: [role1 | A role1 task] **************************************************
    skipping: [abdul]

    TASK: [role2 | A role2 task] **************************************************
    changed: [abdul]

    PLAY RECAP ********************************************************************
    abdul                      : ok=2    changed=1    unreachable=0    failed=0

The _common_ role is skipped, obviously because _role1_ is skipped, and _common_ is a dependency of _role1_.
