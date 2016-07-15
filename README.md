Route Control Role for EOS
==========================

The arista.eos-route-control role creates an abstraction for common EOS
routing policy configuration. This means that you do not need to write any
ansible tasks. Simply create an object that matches the requirements below
and this role will ingest that object and perform the necessary configuration.

This role is used to configure Route-maps and static IPv4 routes.

Installation
------------

```
ansible-galaxy install arista.eos-route-control
```


Requirements
------------

Requires an SSH connection for connectivity to your Arista device. You can use
any of the built-in eos connection variables, or the convenience ``provider``
dictionary.

Role Variables
--------------

The tasks in this role are driven by the ``routemaps`` and
``ipv4_static_routes`` objects described below:

**routemaps** (list) each entry contains the following keys:

|         Key | Type                               | Notes                                                                                                                                                                           |
|------------:|------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|        name | string (required)                  | The name of the routemap to manage.                                                                                                                                             |
|      action | choices:  permit, deny  (required) | The action associated with the routemap name.                                                                                                                                   |
|       seqno | int (required)                     | The sequence number of the rule that this entry corresponds to.                                                                                                                 |
| description | string                             | The description for this routemap entry.                                                                                                                                        |
|       match | list                               | The list of match statements that define the routemap entry. The match statements should be a list of match statements without the word 'match' at the beginning of the string. |
|         set | list                               | The list of set statements that define the routemap entry. The set statements should be a list of set statements without the word 'set' at the beginning of the string.         |
|    continue | int                                | The statement defines the next routemap clause to evaluate.                                                                                                                     |
|       state | choices: present*, absent          | Set the state for the routemap configuration.                                                                                                                                   |


**ipv4_static_routes** (list) each entry contains the following keys:

|         Key | Type                      | Notes                                                                          |
|------------:|---------------------------|--------------------------------------------------------------------------------|
|     ip_dest | string (required)         | Destination IP address or network.                                             |
|    next_hop | string (required)         | The next hop associated with the route.                                        |
| next_hop_ip | string                    | IP address of the next router. Only valid when next_hop is an egress interface |
|    distance | int                       | Distance designated for this route. Defaults to 1 if state is 'present'.       |
|         tag | int                       | Tag assigned for the route. Defaults to 0 if state is 'present'.               |
|  route_name | string                    | Descriptive name for the route                                                 |
|       state | choices: present*, absent | Set the state for the route configuration.                                     |

```
Note: Asterisk (*) denotes the default value if none specified
```

Configuration Variables
-----------------------

|                     Key | Choices      | Description                              |
| ----------------------: | ------------ | ---------------------------------------- |
| eos_save_running_config | true*, false | Specifies whether to write any changes to the running-config resulting from the role execution to memory, copying the configuration to the startup-config. |

```
Note: Asterisk (*) denotes the default value if none specified
```

Connection Variables
--------------------

Ansible EOS roles require the following connection information to establish
communication with the nodes in your inventory. This information can exist in
the Ansible group_vars or host_vars directories, or in the playbook itself.

|         Key | Required | Choices    | Description                              |
| ----------: | -------- | ---------- | ---------------------------------------- |
|        host | yes      |            | Specifies the DNS host name or address for connecting to the remote device over the specified *transport*. The value of *host* is used as the destination address for the transport. |
|        port | no       |            | Specifies the port to use when building the connection to the remote device. This value applies to either acceptable value of *transport*. The port value will default to the appropriate transport common port if none is provided in the task (cli=22, http=80, https=443). |
|    username | no       |            | Configures the usename to use to authenticate the connection to the remote device.  The value of *username* is used to authenticate either the CLI login or the eAPI authentication depending on which *transport* is used. If the value is not specified in the task, the value of environment variable ANSIBLE_NET_USERNAME will be used instead. |
|    password | no       |            | Specifies the password to use to authenticate the connection to the remote device. This is a common argument used for either acceptable value of *transport*. If the value is not specified in the task, the value of environment variable ANSIBLE_NET_PASSWORD will be used instead. |
| ssh_keyfile | no       |            | Specifies the SSH keyfile to use to authenticate the connection to the remote device. This argument is only used when *transport=cli*. If the value is not specified in the task, the value of environment variable ANSIBLE_NET_SSH_KEYFILE will be used instead. |
|   authorize | no       | yes, no*   | Instructs the module to enter priviledged mode on the remote device before sending any commands. If not specified, the device will attempt to excecute all commands in non-priviledged mode. If the value is not specified in the task, the value of environment variable ANSIBLE_NET_AUTHORIZE will be used instead. |
|   auth_pass | no       |            | Specifies the password to use if required to enter privileged mode on the remote device.  If *authorize=no*, then this argument does nothing. If the value is not specified in the task, the value of environment variable ANSIBLE_NET_AUTH_PASS will be used instead. |
|   transport | yes      | cli*, eapi | Configures the transport connection to use when connecting to the remote device. The *transport* argument supports connectivity to the device over cli (ssh) or eapi. |
|     use_ssl | no       | yes*, no   | Configures the transport to use SSL if set to true only when *transport=eapi*.  If *transport=cli*, this value is ignored. |
|    provider | no       |            | Convience method that allows all the above connection arguments to be passed as a dict object. All constraints (required, choices, etc) must be met either by individual arguments or values in this dict. |

```
Note: Asterisk (*) denotes the default value if none specified
```

Ansible Variables
-----------------

|    Key | Choices      | Description                              |
| -----: | ------------ | ---------------------------------------- |
| no_log | true, false* | Prevents module arguments and output from being logged during the playbook execution. By default, no_log is set to true for tasks that gather and save EOS configuration information to reduce output size. Set to true to prevent all output other than task results. |

```
Note: Asterisk (*) denotes the default value if none specified
```


Dependencies
------------

The eos-route-control role is built on modules included in the core Ansible code.
These modules were added in ansible version 2.1

- Ansible 2.1.0

Example Playbook
----------------

The following example will use the arista.eos-route-control role to configure
a route-map and a static route. We'll create a ``hosts`` file with our switch,
then a corresponding ``host_vars`` file and then a simple playbook which only
references the eos-route-control role.
By including the role, we automatically get access to all of the tasks
to configure these EOS features. What's nice about this is that if you have a
host without any corresponding configuration, the tasks will be skipped
without any issue.


Sample hosts file:

    [leafs]
    leaf1.example.com

Sample host_vars/leaf1.example.com

    provider:
      host: "{{ inventory_hostname }}"
      username: admin
      password: admin
      use_ssl: no
      authorize: yes
      transport: cli

    routemaps:
      - name: RM-1
        action: permit
        seqno: 10
        description: My wonderful routemap
        match:
          - as 1000
          - source-protocol bgp
        continue: 20
      - name: RM-1
        action: permit
        seqno: 20
        description: My wonderful routemap
        set:
          - distance 50
          - tag 100

    ipv4_static_routes:
      - ip_dest: 0.0.0.0/0
        next_hop: Management1
        next_hop_ip: 172.16.130.2
        route_name: Default
        tag: 100


A simple playbook, leaf.yml

    - hosts: leafs
      roles:
        - arista.eos-route-control

Then run with:

    ansible-playbook -i hosts leaf.yml
​    


Developer Information
------------

Development contributions are welcome. Please see *Arista Roles for Ansible - Development Guidelines* ([test/arista-ansible-role-test/README](test/arista-ansible-role-test/README.md)) for additional information, including how to develop and run test cases for role development.




License
-------

Copyright (c) 2015, Arista Networks EOS+
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice, this
  list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright notice,
  this list of conditions and the following disclaimer in the documentation
  and/or other materials provided with the distribution.

* Neither the name of Arista nor the names of its
  contributors may be used to endorse or promote products derived from
  this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.


Author Information
------------------

Please raise any issues using our GitHub repo or email us at ansible-dev@arista.com
