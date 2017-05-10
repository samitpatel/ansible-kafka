Kafka
=========
This is the Ansible role for Kafka installation

## Dependencies
- ansible >= 2.0
- zookeeper (https://github.com/samitpatel/ansible-zookeeper)

## Usage
```
#requirements.yml
---

- src: https://github.com/samitpatel/ansible-kafka.git
  name: kafka

#/bin/bash
> ansible-galaxy install -r requirements.yml

#playbook.yml
---
- hosts: all
  become: true
  become_method: sudo
  serial: 1
  roles:
    - kafka

#/bin/bash
> ansible-playbook -i inventory playbook.yml --tags deploy

Available tags:
deploy - Install, configures and starts kafka broker
restart - Restarts kafka broker
configure - Configures kafka broker
stop - Stops kafka broker
```


## Security Features

Kafka security features includes **Encryption**, **Authentication** and **Authorization**. The examples below will show how to enable these features with setting proper ansible variables.

### Encryption and Authentication both DISABLED.
This is the default behavior.

```
- hosts: all
  become: true
  become_method: sudo
  serial: 1
  roles:
    - kafka
```

### Encryption ENABLED and Authentication DISABLED

```
- hosts: all
  become: true
  become_method: sudo
  serial: 1
  roles:
    - kafka
  vars:
    - kafka_security_inter_broker_protocol: SSL
    - kafka_ssl_enabled: true
    - kafka_ssl_endpoint_identification_algorithm: HTTPS
    - kafka_ssl_client_auth: required
    - kafka_ssl_keystore_location: '<path-to-keystore-file>'
    - kafka_ssl_keystore_password: '<keystore-password>'
    - kafka_ssl_key_password: '<key-password>'
    - kafka_ssl_truststore_location: '<path-to-trustore-file>'
    - kafka_ssl_truststore_password: '<truststore-password>'
    - kafka_listeners: 'SSL://{{inventory_hostname}}:9093'
```

### Encryption DISABLED and Authentication ENABLED

```
- hosts: all
  become: true
  become_method: sudo
  serial: 1
  roles:
    - kafka
  vars:
    - kafka_security_inter_broker_protocol: SASL_PLAINTEXT
    - kafka_sasl_enabled: true
    - kafka_sasl_mechanism_inter_broker_protocol: PLAIN
    - kafka_sasl_enabled_mechanisms: PLAIN
    - kafka_super_users: 'User:admin'
    - zookeeper_set_acl: true
    - kafka_admin_password: 'admin-secret'
    - kafka_users:
      - foo: 'bar'
    - kafka_opts: '-Djava.security.auth.login.config={{kafka_conf_dir}}/kafka_server_jaas.conf'
    - kafka_listeners: 'SASL_PLAINTEXT://{{inventory_hostname}}:9094'
```
The example above assumes we have a user 'admin' with all permissions to read/write and communicate with zookeeper and other brokers.

### Encryption and Authentication both ENABLED

```
- hosts: all
  become: true
  become_method: sudo
  serial: 1
  roles:
    - kafka
  vars:
    - kafka_security_inter_broker_protocol: SASL_SSL
    - kafka_ssl_enabled: true
    - kafka_ssl_endpoint_identification_algorithm: HTTPS
    - kafka_ssl_client_auth: required
    - kafka_ssl_keystore_location: '<path-to-keystore-file>'
    - kafka_ssl_keystore_password: '<keystore-password>'
    - kafka_ssl_key_password: '<key-password>'
    - kafka_ssl_truststore_location: '<path-to-trustore-file>'
    - kafka_ssl_truststore_password: '<truststore-password>'
    - kafka_sasl_enabled: true
    - kafka_sasl_mechanism_inter_broker_protocol: PLAIN
    - kafka_sasl_enabled_mechanisms: PLAIN
    - zookeeper_set_acl: true
    - kafka_super_users: 'User:admin'
    - kafka_admin_password: 'admin-secret'
    - kafka_users:
      - foo: 'bar'
      - alice-d-producer: 'alice-secret'
      - bob-d-consumer: 'bob-secret'
    - kafka_opts: '-Djava.security.auth.login.config={{kafka_conf_dir}}/kafka_server_jaas.conf'
    - kafka_listeners: 'SASL_SSL://{{inventory_hostname}}:9095'
```

There is an additional property that defines whether the clients are allowed to access a topic if no ACLs are defined for that topic. By default, this is disabled i.e. set to false. But you enable is by setting the property to true:
`
kafka_allow_everyone_if_no_acl_found: true
`


### Sample JAAS config:
Broker:
```
KafkaServer {
  org.apache.kafka.common.security.plain.PlainLoginModule required

  // Users used for inter-broker-communication
  username="admin"
  password="admin-secret"

  // User definitions
  user_admin="admin-secret"
  user_foo="bar"
  user_alice-d-producer="alice-secret"
  user_bob-d-consumer="bob-secret";
};

// Zookeeper Client authentication
Client {
  org.apache.kafka.common.security.plain.PlainLoginModule required
  username="zookeeper"
  password="zookeeper-password";
};
```

Client:
```
KafkaClient {
  org.apache.kafka.common.security.plain.PlainLoginModule required
  username="foo"
  password="bar";
};
```
