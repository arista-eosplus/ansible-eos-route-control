Route Control Role for EOS
==========================

The arista.eos-route-control role creates an abstraction for common EOS
routing policy configuration. This means that you do not need to write any ansible tasks. Simply create an object that matches the requirements below
and this role will ingest that object and perform the necessary configuration.

This role is used to configure ACLs, Route-maps as well as static IPv4 routes.

Installation
------------

```
ansible-galaxy install arista.eos-route-control
```


Requirements
------------

Requires the arista.eos role.  If you have not worked with the arista.eos role,
consider following the [Quickstart][quickstart] guide.

Role Variables
--------------

The tasks in this role are driven by the ``routemaps``, ``acls`` and
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


**acls** (list) each entry contains the following keys:

|          Key | Type                      | Notes                                                                                                                                                                           |
|-------------:|---------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|         name | string (required)         | The name of the ACL to manage.                                                                                                                                                  |
|       action | string (required)         | The action associated with the ACL name.                                                                                                                                        |
|        seqno | int (required)            | The sequence number of the rule that this entry corresponds to.                                                                                                                 |
|      acltype | string (required)         | The type of ACL to manage. Currently the only supported value for acltype is 'standard'                                                                                         |
|      srcaddr | string (required)         | The list of match statements that define the routemap entry. The match statements should be a list of match statements without the word 'match' at the beginning of the string. |
| srcprefixlen | string (required)         | The statement defines the next routemap clause to evaluate.                                                                                                                     |
|          log | boolean: true, false*     | Enables or disables the log keyword                                                                                                                                             |
|        state | choices: present*, absent | Set the state for the ACL configuration.                                                                                                                                        |

**ipv4_static_routes** (list) each entry contains the following keys:

|         Key | Type                      | Notes                                                                          |
|------------:|---------------------------|--------------------------------------------------------------------------------|
|     ip_dest | string (required)         | Destination IP address or network.                                             |
|    next_hop | string (required)         | The next hop associated with the route.                                        |
| next_hop_ip | string                    | IP address of the next router. Only valid when next_hop is an egress interface |
|    distance | int                       | Distance designated for this route                                             |
|  route_name | string                    | Descriptive name for the route                                                 |
|         tag | int                       | Tag assigned for the route                                                     |
|       state | choices: present*, absent | Set the state for the route configuration.                                     |




```
Note: Asterisk (*) denotes the default value if none specified
```


Dependencies
------------

The eos-route-control role utilizes modules distributed within the
arista.eos role.

- arista.eos version 1.2.0

Example Playbook
----------------

The following example will use the arista.eos-route-control role to configure
a route-map, ACL and a static route. We'll create a
``hosts`` file with our switch, then a corresponding ``host_vars`` file and
then a simple playbook which only references the eos-route-control role.
By including the role, we automatically get access to all of the tasks
to configure these EOS features. What's nice about this is that if you have a
host without any corresponding configuration, the tasks will be skipped
without any issue.


Sample hosts file:

    [leafs]
    leaf1.example.com

Sample host_vars/leaf1.example.com

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

    acls:
      - name: ACL-1
        action: permit
        seqno: 50
        description: My wonderful acl
        type: standard
        srcaddr: 10.10.10.10
        srcprefixlen: 32

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

[quickstart]: http://ansible-eos.readthedocs.org/en/latest/quickstart.html
