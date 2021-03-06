Ansible Role: postgresql-instance
=================================

This role is used for PostgreSQL instance (cluster) management.

Supported OS:
-------------
* RedHat
* CentOS
* OracleLinux

Requirements
------------

None

Role Variables
--------------

Available variables are listed below, along with default values (see `defaults/main.yml`):

    #postgresql_instance_update_etc_pgtab: true
    postgresql_instance_update_etc_pgtab: false

    postgresql_instance_reset: false

    # this option specifies whether or not pg_xlog/pg_wal under PGDATA should point to {{ postgresql_datavg_mountpoint_prefix }}/<instance_name>/wals
    postgresql_instance_create_wal_directory_symlink: true

    postgresql_instance_create_self_signed_certificate: true

    postgresql_instance_certificate_country: PL
    postgresql_instance_certificate_state: 'Pomorskie'
    postgresql_instance_certificate_locality: Gdansk
    postgresql_instance_certificate_organization: 'Foo'
    postgresql_instance_certificate_organizational_unit: IT

    postgresql_instance_backup_user: "{{ postgresql_backup_user|default('backup',true) }}"
    postgresql_instance_nagios_user: "{{ postgresql_nagios_user|default('nagios',true) }}"

    #
    # Software distribution
    #

    # Software location
    postgresql_instance_sw_binary_db:
      - { filename: "{{ postgresql_binary_packages_directory| default( '/software/postgres/binary', true) }}/{{ postgresql_pghomes_binary_prefix| default( 'binary-',true) }}postgresql-9.6.10.tgz", version: 9.6.10 }

    # whether or not 'postgresql_instance_sw_binary_db' is available remotely via NFS
    postgresql_instance_sw_remote_src: no

    postgresql_instance_configure_hba_entries: true

    # Host based authentication (hba) entries to be added to the pg_hba.conf
    postgresql_instance_global_hba_entries:
      - { type: local, database: all, user: postgres, address: '', auth_method: peer, comment: '"local" is for Unix domain socket connections only' }    
      - { type: host, database: all, user: all, address: '127.0.0.1/32', auth_method: md5, comment: "IPv4 local connections:" }
      - { type: host, database: all, user: all, address: '::1/128', auth_method: md5, comment: "IPv6 local connections:" }
      - { type: host, database: all, user: '{{ postgresql_instance_nagios_user }}', address: '{{ansible_fqdn}}', auth_method: md5, comment: "Connections for Nagios monitoring" }
      - { type: host, database: all, user: '{{ postgresql_instance_backup_user }}', address: '{{ansible_fqdn}}', auth_method: trust, comment: "Connections for pg_basebackup" }
      - { type: host, database: replication, user: '{{ postgresql_instance_backup_user }}', address: '{{ansible_fqdn}}', auth_method: trust, comment: "Connections for pg_basebackup" }
      - { type: host, database: all, user: all, address: '192.168.1.0/24', auth_method: 'ldap ldapserver=ldap.foo.com ldapport=636 ldaptls=1 ldapbasedn="o=foo,c=an" ldapsearchattribute=alias', comment: 'LDAP authentication for a named user' }
      - { type: host, database: all, user: all, address: '192.168.24.0/24', auth_method: 'ldap ldapserver=ldap.foo.com ldapport=636 ldaptls=1 ldapbasedn="o=foo,c=an" ldapsearchattribute=alias', comment: 'LDAP authentication for a named user' }

    # set to 'false' - otherwise pg_rewind fails
    postgresql_instance_set_postgresql_conf_read_only: false

    postgresql_instance_configure_postgresql_auto_conf: true

    postgresql_instance_global_postgresql_auto_conf:
      archive_command: "'cp %p {{ postgresql_datavg_mountpoint_prefix }}/archive/{{ item.name }}/%f'"
      archive_mode: "'on'"
      archive_timeout: '10800'
      checkpoint_completion_target: '0.9'
      client_min_messages: "'WARNING'"
      cluster_name: "'{{ item.name }}'"
      fsync: "'on'"
      full_page_writes: "'on'"
      hot_standby_feedback: "'on'"
      hot_standby: "'on'"
      listen_addresses: "'{{inventory_hostname}}'"
      log_autovacuum_min_duration: "'250ms'"
      log_checkpoints: "'on'"
      log_connections: "'on'"
      log_directory: "'{{ postgresql_binvg_mountpoint_prefix }}/diag/{{ item.name }}'"
      log_disconnections: "'off'"
      log_duration: "'off'"
      log_filename: "'postgresql-%a.log'"
      logging_collector: "'on'"
      log_line_prefix: "'%m [%p]: [%l-line] user=%u,db=%d,remote=%h,app=%a '"
      log_lock_waits: "'on'"
      log_min_duration_statement: "'1min'"
      log_min_error_statement: "'error'"
      log_min_messages: "'WARNING'"
      log_rotation_age: 1440
      log_statement: "'ddl'"
      log_temp_files: '0'
    #  log_timezone: "'GMT'"
      log_truncate_on_rotation: "'on'"
      maintenance_work_mem: "'512MB'"
      max_connections: '400'
      max_parallel_workers_per_gather: '4'
      max_replication_slots: '10'
      max_wal_senders: '10'
      max_wal_size: "'1500MB'"
      max_worker_processes: '15'
      pg_stat_statements.track: "'all'"
      port: '{{ item.port }}'
      shared_buffers: "'1024MB'"
      shared_preload_libraries: "'pg_stat_statements'"
      ssl: "'on'"
      synchronous_commit: "'on'"
      tcp_keepalives_count: '9'
      tcp_keepalives_idle: '1800'
      tcp_keepalives_interval: '75'
      temp_buffers: "'64MB'"
      track_activities: "'on'"
      track_activity_query_size: 2048
      track_counts: "'on'"
      track_functions: "'all'"
      wal_compression: "'on'"
      wal_level: "'replica'"
      wal_log_hints: "'on'"
      work_mem: "'64MB'"

    #
    # HA config
    #

    postgresql_instance_configure_recovery_conf: true
    postgresql_instance_switchover_trigger_file: "{{ postgresql_switchover_trigger_file | default('switchover.trigger.file',true) }}"
	  

Dependencies
------------

This role uses `postgresql` role.

Example Playbook
----------------

    - name: Deploy PostgreSQL instance (cluster)
      hosts: pg-servers
      become: true
      become_user: '{{ postgresql_user }}'
      roles:
        - { role: postgresql-instance, tags: postgresql-instance }

Inside `vars/main.yml` or `group_vars/..` or `host_vars/..`:

    #------------------------------------------------
    # overrides role 'postgresql-instance' variables
    #------------------------------------------------

    # Software location
    postgresql_instance_sw_binary_db:
      - { filename: '/software/postgres/binary/foo-postgresql-9.6.10.tgz', version: 9.6.10 }

    postgresql_instance_certificate_country: PL
    postgresql_instance_certificate_state: 'Pomorskie'
    postgresql_instance_certificate_locality: Gdansk
    postgresql_instance_certificate_organization: 'Foo'
    postgresql_instance_certificate_organizational_unit: IT

    # Host based authentication (hba) entries to be added to the pg_hba.conf
    postgresql_instance_global_hba_entries:
      - { type: local, database: all, user: postgres, address: '', auth_method: peer, comment: '"local" is for Unix domain socket connections only' }
    #  - { type: local, database: all, user: all, address: '', auth_method: peer, comment: '"local" is for Unix domain socket connections only' }
      - { type: host, database: all, user: all, address: '127.0.0.1/32', auth_method: md5, comment: "IPv4 local connections:" }
      - { type: host, database: all, user: all, address: '::1/128', auth_method: md5, comment: "IPv6 local connections:" }
      - { type: host, database: all, user: '{{ postgresql_instance_nagios_user }}', address: '{{ansible_fqdn}}', auth_method: md5, comment: "Connections for Nagios monitoring" }
      - { type: host, database: all, user: '{{ postgresql_instance_backup_user }}', address: '{{ansible_fqdn}}', auth_method: trust, comment: "Connections for pg_basebackup" }
      - { type: host, database: replication, user: '{{ postgresql_instance_backup_user }}', address: '{{ansible_fqdn}}', auth_method: trust, comment: "Connections for pg_basebackup" }
      - { type: host, database: all, user: pgadmin, address: '0.0.0.0/0', auth_method: md5, comment: "pgadmin connection" }
      - { type: host, database: all, user: all, address: '192.168.1.0/24', auth_method: 'ldap ldapserver=ldap.foo.com ldapport=636 ldaptls=1 ldapbasedn="o=foo,c=an" ldapsearchattribute=alias', comment: 'LDAP authentication for a named user' }
      - { type: host, database: all, user: all, address: '192.168.24.0/24', auth_method: 'ldap ldapserver=ldap.foo.com ldapport=636 ldaptls=1 ldapbasedn="o=foo,c=an" ldapsearchattribute=alias', comment: 'LDAP authentication for a named user' }


    # ... etc ...


License
-------

GPLv3 - GNU General Public License v3.0

Author Information
------------------

This role was created in 2018 by [Krzysztof Lewandowski](mailto:Krzysztof.Lewandowski@fastmail.fm).


