Role Name
=========

Ansible role to install the [operator-framework](https://github.com/operator-framework/getting-started) sdk on Fedora.

It also installs kubectl and minikube as your container orchestration engine.

Requirements
------------

You'll need to list your dependencies for the [virtualization driver](https://github.com/kubernetes/minikube/blob/master/docs/drivers.md) minikube to consume. As well as the [checksum](https://github.com/kubernetes/minikube/releases) and [version](https://github.com/kubernetes/minikube/releases). 

Role Variables
--------------
user is the only required variable which sets your /home path and adds to the libvirt group to be able to run minikube without having to use sudo.

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
