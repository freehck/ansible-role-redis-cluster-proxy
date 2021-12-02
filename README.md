freehck.redis_cluster_proxy
=========

Deploys redis cluster proxy

Description
-----------

This role deploys the redis cluster proxy, that pretends to be a standalone redis for clients, working in the same time with the redis cluster in the background. Beware: this software is beta. It's not even released yet. So use it on your own risk.

Role Variables
--------------

The most important variables are listed here. For all the variables look into `defaults/main.yml`.

`redis_cluster_proxy_version`: version of the redis-server, default is `1.0-beta2`

`redis_cluster_proxy_install_dir`: redis will be installed here, default is `/opt/redis-cluster-proxy-1.0-beta2`

The only mandatory variable `redis_cluster_proxy_nodes`. The default is `[]`. You can fill it with `redis_cluster_nodes` from the `freehck.redis_cluster` role (look example below). But actually it must be just a list of addresses and ports in format like this:

    redis_cluster_proxy_nodes:
      - addr: 10.1.1.1
        port: 6378
      - addr: 10.1.1.2
        port: 6378
      <...>

Example
-------

Inventory:

    redis01 ansible_host=10.1.1.1
    redis02 ansible_host=10.1.1.2
    redis03 ansible_host=10.1.1.3

    [redis_all]
    redis0[1:3]

    [redis_proxy]
    redis01

Group vars `group_vars/redis_all.yml`:

    redis_cluster_nodes:

      # masters
      - name: master01
        host: redis01
        addr: "{{ hostvars['redis01'].ansible_host }}"
        port: 6378
        is_master: true

      - name: master02
        host: redis02
        addr: "{{ hostvars['redis02'].ansible_host }}"
        port: 6378
        is_master: true

      - name: master03
        host: redis03
        addr: "{{ hostvars['redis03'].ansible_host }}"
        port: 6378
        is_master: true

      # slaves
      - name: slave02
        host: redis01
        addr: "{{ hostvars['redis01'].ansible_host }}"
        port: 6379
        slave_of: master02

      - name: slave03
        host: redis02
        addr: "{{ hostvars['redis02'].ansible_host }}"
        port: 6379
        slave_of: master03

      - name: slave01
        host: redis03
        addr: "{{ hostvars['redis03'].ansible_host }}"
        port: 6379
        slave_of: master01

    redis_cluster_proxy_bind_addr: "{{ backnet_ip }}"
    redis_cluster_proxy_bind_port: "7777"
    redis_cluster_proxy_nodes: "{{ redis_cluster_nodes }}"

Playbook:

    - hosts:
        - redis_all
      become: true
      roles:
        - role: freehck.redis_cluster

    - hosts:
        - redis_proxy
      become: true
      roles:
        - role: freehck.redis_cluster_proxy



Install
-------

This role can be installed from [Ansible Galaxy](https://galaxy.ansible.com/):

`ansible-galaxy install freehck.redis_cluster_proxy`


License
-------

MIT

Author Information
------------------

Dmitrii Kashin, <freehck@freehck.com>
