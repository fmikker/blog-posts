check_lm_sensors.pl dependencies on CentOS 7:

yum install hddtemp
yum install perl-List-MoreUtils
yum install perl-Monitoring-    check_lm_sensors: created new branch v3.1.0
    
        * modified the plugin to use Monitoring::Plugins instead of Nagios::Plugin
        * modified spec-file to use perl-Monitoring-Plugin
        * changed the shebang to use #!/usr/bin/env perl



