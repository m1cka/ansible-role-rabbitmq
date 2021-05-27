# RabbitMQ Ansible Role
Forked from https://github.com/anthemengineering/ansible-role-rabbitmq on 2021-05-27
to support EL 7 and RabbitMQ 3.7+.

## Version

See:

- [RabbitMQ - Server Releases](https://www.rabbitmq.com/releases/rabbitmq-server/)

Set the `rabbitmq_version` variable to define the version of RabbitMQ to install.

```yaml
rabbitmq_version: '3.8.16'
```

## Users

See:

- [Ansible - RabbitMQ User Module](http://docs.ansible.com/ansible/rabbitmq_user_module.html)
- [RabbitMQ - Access Control](https://www.rabbitmq.com/access-control.html)

Set the `rabbitmq_users` variable to define an array of present users.

```yaml
rabbitmq_users:
  - user: admin
    password: admin
    tags: administrator
```

| parameter      | required | default | choices | comments |
| -------------- | -------- | ------- | ------- | -------- |
| configure_priv | no       | .*      |         |          |
| password       | yes      |         |         |          |
| read_priv      | no       | .*      |         |          |
| tags           | no       |         |         |          |
| user           | yes      |         |         |          |
| vhost          | no       | /       |         |          |
| write_priv     | no       | .*      |         |          |

### Remove Users

Set the `rabbitmq_users_absent` variable to define an array of absent users.

```yaml
rabbitmq_users_absent:
  - guest
```

## Virtual Hosts

See:

- [Ansible - RabbitMQ Virtual Host Module](http://docs.ansible.com/ansible/rabbitmq_vhost_module.html)
- [RabbitMQ - Virtual Hosts](https://www.rabbitmq.com/vhosts.html)

Set the `rabbitmq_vhosts` variable to define an array of present virtual hosts.

```yaml
rabbitmq_vhosts:
  - /one
  - name: /two
    node: rabbit
    tracing: no
```

| parameter  | required | default | choices                          | comments |
| ---------- | -------- | ------- | -------------------------------- | -------- |
| name       | yes      |         |                                  |          |
| node       | no       | rabbit  |                                  |          |
| tracing    | no       | no      | <ul><li>yes</li><li>no</li></ul> |          |

### Remove Virtual Hosts

Set the `rabbitmq_vhosts_absent` variable to define an array of absent virtual hosts.

```yaml
rabbitmq_vhosts_absent:
  - /vhost
```

## Plugins

See:

- [Ansible - RabbitMQ Plugin Module](http://docs.ansible.com/ansible/rabbitmq_plugin_module.html)
- [RabbitMQ - Plugins](https://www.rabbitmq.com/plugins.html)

Set the `rabbitmq_plugins` variable to define an array of enabled plugins.

```yaml
rabbitmq_plugins:
  - rabbitmq_management
  - name: rabbitmq_delayed_message_exchange
    url: http://www.rabbitmq.com/community-plugins/v3.6.x/rabbitmq_delayed_message_exchange-0.0.1.ez
```

| parameter | required | default | choices | comments            |
| --------- | -------- | ------- | ------- | ------------------- |
| name      | yes      |         |         |                     |
| url       | no       |         |         | Installs the plugin |

### Disable Plugins

Set the `rabbitmq_plugins_disabled` variable to disable plugins.

```yaml
rabbitmq_plugins_disabled:
  - rabbitmq_management
```

## Configuration

See:

- [RabbitMQ - Configuration](https://www.rabbitmq.com/configure.html)

Set the `rabbitmq_config` variable to define the configuration.

```yaml
rabbitmq_config:
  listeners.tcp.default: 5672
```

Set the `rabbitmq_env` variable to define the environment variables. Note that the keys should not contain the
"RABBITMQ_" prefix.

```yaml
rabbitmq_env:
  DIST_PORT: 25672
```

## Cluster

See:

- [RabbitMQ - Clustering Guide](https://www.rabbitmq.com/clustering.html)

Set the `rabbitmq_cluster` variable to enable clustering.

As the above clustering documentation is pretty hard to grasp I suggest reading of
[https://computingforgeeks.com/how-to-configure-rabbitmq-cluster-on-ubuntu-18-04-lts/](https://computingforgeeks.com/how-to-configure-rabbitmq-cluster-on-ubuntu-18-04-lts/)
for quick start. And then defined minimum variable as below where the
  `rabbitmq1` is the short hostname of the master node.

```yaml
rabbitmq_cluster: yes
# shortname dns only
rabbitmq_cluster_master: "rabbit@rabbitmq1"
```

Please note that the default behaviour is:
- The first node of the host group is the master
- The ha policy is to replicate the queue for all nodes
- Replacing non-master nodes is supported - just destroy the non-master node and
  re-launch.
- Switching master node to other node is not supported unless you have to do it
  manually or rebuild the whole cluster.

### Erlang Cookie

Set the `rabbitmq_erlang_cookie` variable to define the Erlang cookie.

```yaml
rabbitmq_erlang_cookie: g9avtqdzdm2p5oe9
```

## License

MIT
