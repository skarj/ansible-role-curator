Elasticsearch Curator role
=========
[![Build Status](https://api.travis-ci.org/skarj/ansible-role-curator.svg?branch=master)](https://travis-ci.org/skarj/ansible-role-curator)

Ansible role for [elasticsearch curator](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/about.html) 5.x installation.

Requirements
------------

* Ansible 2.5
* This role requires [hash_behaviour=merge](http://docs.ansible.com/ansible/latest/reference_appendices/config.html#default-hash-behaviour) for variables


Dependencies
------------

None


Role Variables
--------------

Default config variables are described in *defaults/main.yml*.
If you need an extended configuration you can specify it in the variables for each environment in section *curator.config*.
Structure of section *curator.config* repeats *curator.yml*, you can insert any parameters described in official curator [documentation](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/configfile.html)


Example Playbook
----------------

An example of how to use elasticsearch server role:

        - hosts: all
          roles:
            - role: ansible-role-curator
              curator:
                # Actions are the tasks which Curator can perform on your indices
                action_configs:
                  # Name for action configuration file
                  delete_indices:
                    # Options for action configuarion file
                    # Remember, leave a key empty if there is no value.  None will be a string,
                    # not a Python "NoneType"
                    actions:
                      1:
                        action: delete_indices
                        description: "Delete filebeat indices older than 7 days (not related to specific projects and environments)"
                        filters:
                          - filtertype: pattern
                            kind: regex
                            value: '^filebeat-\d{1}.\d{1}.\d{1}-\d{4}.\d{2}.\d{2}'
                            exclude:
                          - filtertype: age
                            source: creation_date
                            direction: older
                            timestring: '%Y.%m.%d'
                            unit: days
                            unit_count: 7
                            exclude:
                        options:
                          continue_if_exception: false
                          disable_action: false
                          ignore_empty_list: true
                          timeout_override:

                      2:
                        action: delete_indices
                        description: Delete logstash indices older than 7 days (not related to specific projects and environments)"
                        filters:
                          - filtertype: pattern
                            kind: regex
                            value: '^logstash-\d{4}.\d{2}.\d{2}'
                            exclude:
                          - filtertype: age
                            source: creation_date
                            direction: older
                            timestring: '%Y.%m.%d'
                            unit: days
                            unit_count: 7
                            exclude:
                        options:
                          continue_if_exception: false
                          disable_action: false
                          ignore_empty_list: true
                          timeout_override:
                  # Name for action configuration file
                  create_snapshot:
                    # Options for action configuarion file
                    # Remember, leave a key empty if there is no value. None will be a string,
                    # not a Python "NoneType"
                    actions:
                      1:
                        action: snapshot
                        description: Snapshot logstash- prefixed indices older than 1 day
                        options:
                          repository:
                          # Leaving name blank will result in the default 'curator-%Y%m%d%H%M%S'
                          name:
                          ignore_unavailable: False
                          include_global_state: True
                          partial: False
                          wait_for_completion: True
                          skip_repo_fs_check: False
                          timeout_override:
                          continue_if_exception: False
                          disable_action: False
                        filters:
                        - filtertype: pattern
                          kind: prefix
                          value: logstash-
                          exclude:
                        - filtertype: age
                          source: creation_date
                          direction: older
                          unit: days
                          unit_count: 1
                          exclude:

                cron_jobs:
                  - name: "Delete old elasticsearch indices"
                    job: "/opt/elasticsearch-curator/curator --config /etc/curator/curator.yml /etc/curator/actions/delete_indices.yml"
                    minute: 0
                    hour: 1
                  - name: "Create index snapshot"
                    job: "/opt/elasticsearch-curator/curator --config /etc/curator/curator.yml /etc/curator/actions/create_snapshot.yml"
                    minute: 0
                    hour: 0


License
-------

MIT


Author Information
------------------

Sergey Baranov <skaarj.sergey@gmail.com>
