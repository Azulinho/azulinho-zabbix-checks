This repo contains ansible code to configure monitoring items on a zabbix server

When cloning from github, simply run:

    rake

When using galaxy, simply run:

    ansible-galaxy install Azulinho.azulinho-zabbix-checks

To consume this role, add the following variables to group_vars/all or to
a wrapper_role <wrapper_role/vars/main.yaml>

    azulinho_zabbix_checks:
        nodeid: 999
        zabbix_username: admin
        zabbix_password: zabbix
        zabbix_host: localhost
        mysql_username: zabbix
        mysql_password: password
        mysql_port: 3306
        mysql_start_pollers: 25
        mysql_start_trappers: 25
        server_port: 10051
        client_port: 10050
        dbname: zabbix
        dbhost: localhost

        # List of all the zabbix templates applied per hostgroup
        # zabbix hostgroups match the ansible group for the box
        # the host_group 'all_servers' contain a list of zabbix templates that is
        # common and applied to every box
        #
        #
        host_groups: &zabbix_host_groups_defaults
          all_servers:
            templates: "OS_Linux,App_SSH_Service,App_Zabbix_Agent"
          vagrant_servers:
            templates: "HTTP,mysql"


        # zabbix_xml_templates contains the zabbix definitions for templates, this
        # block builds the templates using jinja2 templating, which will than be
        # applied to a group of hosts based on the zabbix dictionary
        #
        #
        #
        #
        xml_templates: &zabbix_xml_templates_defaults
          OS_Linux:
            applications: ['CPU', 'Filesystems', 'General', 'Memory', 'Network interfaces', 'OS', 'Performance', 'Processes', 'Security', 'test1' ]
            zabbix_items:
              - { name: 'Available memory', xkey: 'vm.memory.size[available]', value_type: 3, units: 'B', description: 'Available memory is defined as free+cached+buffers memory', applications: ['Memory'] }
              - { name: 'Checksum of $1', xkey: 'vfs.file.cksum[/etc/passwd]', delay: 3600, value_type: 3, units: 'B', applications: ['Security'] }
              - { name: 'Context switches per second', xkey: 'system.cpu.switches', value_type: 3, units: 'sps', delta: 1, applications: ['CPU', 'Performance'] }
              - { name: 'CPU $2 time', xkey: 'system.cpu.util[,nice]',units: '%', description: 'The time the CPU has spent running users process that have been niced', applications: ['CPU', 'Performance'] }
              - { name: 'CPU $2 time', xkey: 'system.cpu.util[,interrupt]', units: '%', description: 'The time the CPU has been servicing hardware interrupts', applications: ['CPU', 'Performance'] }
              - { name: 'CPU $2 time', xkey: 'system.cpu.util[,softirq]',  units: '%', description: 'The time the CPU has been servicing software interrupts', applications: ['CPU', 'Performance'] }
              - { name: 'CPU $2 time', xkey: 'system.cpu.util[,system]', units: '%', description: 'The time the CPU has spent running the kernel and its processes', applications: ['CPU', 'Performance'] }
              - { name: 'CPU $2 time', xkey: 'system.cpu.util[,user]', units: '%', description: 'The time the CPU has spent running users processes', applications: ['CPU', 'Performance'] }
              - { name: 'CPU $2 time', xkey: 'system.cpu.util[,idle]', units: '%', description: 'The time the CPU has spent doing nothing', applications: ['CPU', 'Performance'] }
              - { name: 'CPU $2 time', xkey: 'system.cpu.util[,steal]', units: '%', description: 'The amount of CPU stolen by the hypervisor for other VMs', applications: ['CPU', 'Performance'] }
              - { name: 'CPU $2 time', xkey: 'system.cpu.util[,iowait]', units: '%', description: 'The amount of time the CPU has been waiting for IO to complete', applications: ['CPU', 'Performance'] }
              - { name: 'Host boot time', xkey: 'system.boottime', value_type: 3, delay: 600, units: 'unixtime', applications: ['General', 'OS'] }
              - { name: 'Host local time', xkey: 'system.localtime', value_type: 3, units: 'unixtime', applications: ['General', 'OS'] }
              - { name: 'Host name', xkey: 'system.hostname', delay: 3600, value_type: 1, description: 'System host name', inventory_link: 3, applications: ['General', 'OS'] }
              - { name: 'Interrupts per second', xkey: 'system.cpu.intr',value_type: 3, units: 'ips', delta: 1, applications: ['CPU', 'Performance'] }
              - { name: 'Maximum number of opened files', xkey: 'kernel.maxfiles', delay: 3600, value_type: 3, units: 'ips', description: 'Maximum number of open files', applications: ['OS'] }
              - { name: 'Maximum number of processes', xkey: 'kernel.maxproc', delay: 3600, value_type: 3, description: 'Maximum number of processes', applications: ['OS'] }
              - { name: 'Number of logged in users', xkey: 'system.users.num',value_type: 3, description: 'Number of users who are currently logged in', applications: ['OS', 'Security'] }
              - { name: 'Number of processes', xkey: 'proc.num[]', value_type: 3, description: 'Total number of processes in any state', applications: ['Processes'] }
              - { name: 'Number of running processes', xkey: 'proc.num[,,run]', value_type: 3, description: 'Total number of processes in running state', applications: ['Processes'] }
              - { name: 'Processor load(1min average per core)', xkey: 'system.cpu.load[percpu,avg1]' ,value_type: 0, description: 'Processor Load', applications: ['CPU', 'Performance'] }
              - { name: 'Processor load(5min average per core)', xkey: 'system.cpu.load[percpu,avg5]' ,value_type: 0, description: 'Processor Load', applications: ['CPU', 'Performance'] }
              - { name: 'Processor load(15min average per core)', xkey: 'system.cpu.load[percpu,avg15]' ,value_type: 0, description: 'Processor Load', applications: ['CPU', 'Performance'] }
              - { name: 'System information', xkey: 'system.uname', delay: 3600, value_type: 1, description: 'the info as returned by uname -a', applications: ['General', 'OS'] }
              - { name: 'System uptime', xkey: 'system.uptime', delay: 600, value_type: 3, applications: ['General', 'OS'] }
              - { name: 'Total memory', xkey: 'vm.memory.size[total]', delay: 3600, value_type: 3, units: 'B', applications: ['Memory'] }
            discovery_rules:
              - { name: 'Mounted filesystem discovery', xkey: 'vfs.fs.discovery', filter: '{#FSTYPE}:@File systems for discovery', lifetime: 30, description: 'Discovery of filesystems',
                  item_prototypes: [
                    {name: 'Free disk space on $1', xkey: 'vfs.fs.size[{#FSNAME},free]', value_type: 3, units: 'B', applications: ['Filesystems']},
                    {name: 'Free disk space on $1 (percentage)', xkey: 'vfs.fs.size[{#FSNAME},pfree]', value_type: 0, units: '%', applications: ['Filesystems']},
                    {name: 'Free inodes on $1 (percentage)', xkey: 'vfs.fs.inode[{#FSNAME},pfree]', value_type: 0, units: '%', applications: ['Filesystems']},
                    {name: 'Total disk space $1 ', xkey: 'vfs.fs.size[{#FSNAME},total]', delay: 3600, value_type: 3, units: 'B', applications: ['Filesystems']},
                    {name: 'Used disk space $1 ', xkey: 'vfs.fs.size[{#FSNAME},used]', value_type: 3, units: 'B', applications: ['Filesystems']} ],
                  triggers_prototypes: [
                    { name: 'Free disk space is less than 20% on volume {#FSNAME}', expression: '{OS_Linux:vfs.fs.size[{#FSNAME},pfree].last(0)}&lt;20', status: 0, priority: 2 },
                    { name: 'Free inodes is less than 20% on volume {#FSNAME}', expression: '{OS_Linux:vfs.fs.inode[{#FSNAME},pfree].last(0)}&lt;20', status: 0, priority: 2 } ],
                  graph_prototypes: [
                    { name: 'Disk space usage {#FSNAME}', type: 2, width: 600, height: 340,
                      graph_items: [
                        { sortorder: 0, color: C80000, type: 2, host: 'OS_Linux', gkey: 'vfs.fs.size[{#FSNAME},total]'},
                        { sortorder: 1, color: 00C800, type: 0, host: 'OS_Linux', gkey: 'vfs.fs.size[{#FSNAME},free]'} ] } ]}
              - { name: 'Network interface discovery', xkey: 'net.if.discovery', delay: 3600, filter: '{#IFNAME}:@Network interfaces for discovery', lifetime: 30, description: 'Discovery of network interfaces',
                  item_prototypes: [
                    {name: 'Incoming network traffic $1', xkey: 'net.if.in[{#IFNAME}]', delay: 60, value_type: 3, units: 'bps', delta: 1, formula: 8, applications: ['Network interfaces']},
                    {name: 'Outgoing network traffic $1', xkey: 'net.if.out[{#IFNAME}]', delay: 60, value_type: 3, units: 'bps', delta: 1, formula: 8, applications: ['Network interfaces']} ],
                  triggers_prototypes: [],
                  graph_prototypes: [
                    { name: 'Network traffic on {#IFNAME}', width: 900, height: 200, show_work_period: 1, show_triggers: 1, type: 0, show_3d: 0,
                      graph_items: [
                      { color: 00AA00, host: 'OS_Linux', gkey: 'net.if.in[{#IFNAME}]', calc_fnc: 2, drawtype: 5 },
                      { color: 3333FF, host: 'OS_Linux', gkey: 'net.if.out[{#IFNAME}]', sortorder: 1, calc_fnc: 2, drawtype: 5} ] } ]}
            screens:
              - { name: 'System performance', hsize: 2, vsize: 3,
                  screen_items: [
                    { width: 500, height: 120, colspan: 1, rowspan: 1, valign: 1, name: 'CPU load', host: 'OS_Linux' },
                    { width: 500, height: 148, x: 0, y: 1, colspan: 1, rowspan: 1, valign: 1, name: 'CPU utilization', host: 'OS_Linux' },
                    { width: 500, height: 100, x: 0, y: 1, colspan: 1, elements: 0, rowspan: 1, valign: 0, name: 'Memory usage', host: 'OS_Linux' }]}
            triggers:
              - { expression: '{OS_Linux:kernel.maxfiles.last(0)}&lt;1024', name: 'Configured max number of opened files is too low on {HOST.NAME}', status: 0, priority: 1, type: 0 }
              - { expression: '{OS_Linux:kernel.maxproc.last(0)}&lt;256', name: 'Configured max number of processes is too low on {HOST.NAME}', status: 0, priority: 1, type: 0 }
              - { expression: '{OS_Linux:system.cpu.util[,iowait].avg(15m)}&gt;20', name: 'Disk IO is overloaded on {HOST.NAME}', status: 0, priority: 2, type: 0 }
              - { expression: '{OS_Linux:system.uname.diff(0)}&gt;0', name: 'Host information was changed on {HOST.NAME}', status: 0, priority: 1, type: 0 }
              - { expression: '{OS_Linux:system.hostname.diff(0)}&gt;0', name: 'Hostname was changed on {HOST.NAME}', status: 0, priority: 1, type: 0 }
              - { expression: '{OS_Linux:vm.memory.size[available].last(0)}&lt;20M', name: 'Lack of available memory on server {HOST.NAME}', status: 0, priority: 3, type: 0 }
              - { expression: '{OS_Linux:system.cpu.load[percpu,avg1].avg(5m)}&gt;5', name: 'Processor load is too high on {HOST.NAME}', status: 0, priority: 2, type: 0 }
              - { expression: '{OS_Linux:proc.num[,,run].avg(5m)}&gt;300', name: 'Too many processes running on {HOST.NAME}', status: 0, priority: 2, type: 0 }
              - { expression: '{OS_Linux:system.uptime.change(0)}&lt;0', name: '{HOST.NAME} has just been restarted', status: 0, priority: 1, type: 0 }
            graphs:
              - { name: 'CPU jumps', width: 900, height: 200,
                  graph_items: [
                    { color: 009900, calc_fnc: 2, host: 'OS_Linux', gikey: 'system.cpu.switches' },
                    { sortorder: 1, color: 000099, calc_fnc: 2, host: 'OS_Linux', gikey: 'system.cpu.intr' }] }
              - { name: 'CPU load', width: 900, height: 200,
                  graph_items: [
                    { sortorder: 1, color: 000099, calc_fnc: 2, host: 'OS_Linux', gikey: 'system.cpu.load[percpu,avg5]' },
                    { sortorder: 2, color: 990000, calc_fnc: 2, host: 'OS_Linux', gikey: 'system.cpu.load[percpu,avg15]' } ]}
              - { name: 'CPU utilization', width: 900, height: 200,
                  graph_items: [
                    { sortorder: 0, drawtype: 1, color: FF5555, calc_fnc: 2, host: 'OS_Linux', gikey: 'system.cpu.util[,steal]' },
                    { sortorder: 1, drawtype: 1, color: 55FF55, calc_fnc: 2, host: 'OS_Linux', gikey: 'system.cpu.util[,softirq]' },
                    { sortorder: 2, drawtype: 1, color: 009999, calc_fnc: 2, host: 'OS_Linux', gikey: 'system.cpu.util[,interrupt]' },
                    { sortorder: 3, drawtype: 1, color: 990099, calc_fnc: 2, host: 'OS_Linux', gikey: 'system.cpu.util[,nice]' },
                    { sortorder: 4, drawtype: 1, color: 999900, calc_fnc: 2, host: 'OS_Linux', gikey: 'system.cpu.util[,iowait]' },
                    { sortorder: 5, drawtype: 1, color: 990000, calc_fnc: 2, host: 'OS_Linux', gikey: 'system.cpu.util[,system]' },
                    { sortorder: 6, drawtype: 1, color: 000099, calc_fnc: 2, host: 'OS_Linux', gikey: 'system.cpu.util[,user]' },
                    { sortorder: 7, drawtype: 1, color: 009900, calc_fnc: 2, host: 'OS_Linux', gikey: 'system.cpu.util[,idle]' } ] }
              - { name: 'Memory usage', width: 900, height: 200, show_legend: 1,
                  graph_items: [
                    { sortorder: 1, drawtype: 5, color: 00C800, calc_fnc: 2, host: 'OS_Linux', gikey: 'vm.memory.size[available]' },
                    { sortorder: 0, drawtype: 0, color: DD0000, calc_fnc: 2, host: 'OS_Linux', gikey: 'vm.memory.size[total]' },
                    { sortorder: 0, drawtype: 0, color: DD0000, calc_fnc: 2, host: 'OS_Linux', gikey: 'vm.memory.size[total]' } ] }

          App_SSH_Service:
            applications: ['SSH service']
            zabbix_items:
              - { name: 'SSH service is running', type: 3, xkey: 'net.tcp.service[ssh]', value_type: 3, applications: ['SSH service'] }
            triggers:
              - { expression: '{App_SSH_Service:net.tcp.service[ssh].max(#3)}=0', name: 'SSH service on {HOST.NAME}', status: 0, priority: 3, type: 0 }

          App_Zabbix_Agent:
            applications: ['Zabbix agent']
            zabbix_items:
              - { name: 'Agent ping', type: 0, xkey: 'agent.ping', value_type: 3, applications: ['Zabbix agent'] }
              - { name: 'Host name of zabbix_agentd running', delay: 3600, type: 0, xkey: 'agent.hostname', value_type: 1, applications: ['Zabbix agent'] }
              - { name: 'Version of zabbix_agent(d) running', delay: 3600, type: 0, xkey: 'agent.version', value_type: 1, applications: ['Zabbix agent'] }
            triggers:
              - { expression: '{App_Zabbix_Agent:agent.hostname.diff(0)}&gt;0', name: 'Host name of zabbix_agentd was changed on {HOST.NAME}', status: 0, priority: 1, type: 0 }
              - { expression: '{App_Zabbix_Agent:agent.version.diff(0)}&gt;0', name: 'Version of zabbix_agentd was changed on {HOST.NAME}', status: 0, priority: 1, type: 0 }
              - { expression: '{App_Zabbix_Agent:agent.ping.nodata(300)}=1', name: 'Zabbix agent on {HOST.NAME} is unreachable for 5 minutes', status: 0, priority: 3, type: 0 }

          Base: {}


          HTTP:
            applications: ['HTTP']
            zabbix_items:
              - { name: 'HTTP_port_check', type: 0, xkey: 'net.tcp.listen[80]', value_type: 3, applications: ['HTTP'] }
            triggers:
              - { expression: '{HTTP:net.tcp.listen[80].max(5m)}=0|({TRIGGER.VALUE}=1 &amp; {HTTP:net.tcp.listen[80].min(5m)} &gt; 0)', name: 'HTTP service on {HOST.NAME}', status: 0, priority: 4, type: 0 }

          mysql:
            applications: ['mysql']
            zabbix_items:
              - { name: 'mysql_port_check',
                  type: 0,
                  xkey: 'net.tcp.listen[3306]',
                  value_type: 3,
                  applications: ['mysql'] }
            triggers:
              - { expression: '{mysql:net.tcp.listen[3306].max(5m)}=0|({TRIGGER.VALUE}=1 &amp; {mysql:net.tcp.listen[3306].min(5m)} &gt; 0)',
                  name: 'mysql service on {HOST.NAME}',
                  status: 0,
                  priority: 4,
                  type: 0 }

        hosts:
          - { hostname: 'vagrantbox' , hostgroup: 'vagrant_servers', ip: '192.168.67.5' }


    azulinho_zabbix_server: "{{ azulinho_zabbix_checks }}"
    azulinho_zabbix_agent: "{{ azulinho_zabbix_checks }}"
