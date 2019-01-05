Role Name
=========

Ansible role to install the [operator-framework](https://github.com/operator-framework/getting-started) sdk on Fedora.

This playbook also installs minikube as your container orchestration engine.

Requirements
------------

Any pre-requisites that may not be covered by Ansible itself or the role should be mentioned here. For instance, if the role uses the EC2 module, it may be a good idea to mention in this section that the boto package is required.

Role Variables
--------------

A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.

Dependencies
------------

Set your defaults.

    ---
    # defaults file for role_install_operator_framework
    driver_url: 'https://storage.googleapis.com/minikube/releases/latest/docker-machine-driver-kvm2'
    mini_checksum: 'sha256:3298d3183deacd9ddd3032dab113a64d863df7648d6d24693284ba4193e95b49'
    mini_version: 'v0.32.0'

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    ---
    # Ansible playbook which installs the operator-framework
    # https://github.com/operator-framework/getting-started
    # Example: ansible-playbook -v pb_install_operator_framework.yaml -e user=your_user_here
    - hosts: 127.0.0.1
      connection: local
      gather_facts: false
      become: True
      become_user: root
    
      roles:
        - { role: role_install_operator_framework }

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
