Nginx status
=====

nginx
-----

	server {
		listen       8089;
		server_name  localhost;

		location /stub_status {
			stub_status on;
			access_log off;
			allow 127.0.0.1;
			deny all;
		}
	}

zabbix_agentd
-----
	cat /etc/zabbix/zabbix_agentd.d/userparameter_nginx.conf

	UserParameter=nginx.status[*],/etc/zabbix/scripts/nginx.sh $1

scripts
-----

	mkdir -p /etc/zabbix/scripts
	vim /etc/zabbix/scripts/nginx.sh

	#!/bin/bash
	##################################################
	# AUTHOR: Anup Dubey >> anup.hnu@outlook.com
	# Description：status of nginx
	##################################################

	HOST="localhost"
	PORT="8089"
	stub_status=stub_status

	function check() {
		if [ -f /sbin/pidof ]; then
		   /sbin/pidof nginx | wc -w
		else
		   ps ax | grep "nginx:" | grep -v grep | wc -l
		fi
	}

	function active() {
		/usr/bin/curl -s "http://$HOST:$PORT/${stub_status}/" 2>/dev/null| grep 'Active' | awk '{print $NF}'
	}
	function accepts() {
		/usr/bin/curl -s "http://$HOST:$PORT/${stub_status}/" 2>/dev/null| awk NR==3 | awk '{print $1}'
	}
	function handled() {
		/usr/bin/curl -s "http://$HOST:$PORT/${stub_status}/" 2>/dev/null| awk NR==3 | awk '{print $2}'
	}
	function requests() {
		/usr/bin/curl -s "http://$HOST:$PORT/${stub_status}/" 2>/dev/null| awk NR==3 | awk '{print $3}'
	}
	function reading() {
		/usr/bin/curl -s "http://$HOST:$PORT/${stub_status}/" 2>/dev/null| grep 'Reading' | awk '{print $2}'
	}
	function writing() {
		/usr/bin/curl -s "http://$HOST:$PORT/${stub_status}/" 2>/dev/null| grep 'Writing' | awk '{print $4}'
	}
	function waiting() {
		/usr/bin/curl -s "http://$HOST:$PORT/${stub_status}/" 2>/dev/null| grep 'Waiting' | awk '{print $6}'
	}

	case "$1" in
		check)
			check
			;;
		active)
			active
			;;
		accepts)
			accepts
			;;
		handled)
			handled
			;;
		requests)
			requests
			;;
		reading)
			reading
			;;
		writing)
			writing
			;;
		waiting)
			waiting
			;;

		*)
			echo $"Usage $0 {check|active|accepts|handled|requests|reading|writing|waiting}"
			exit		
	esac

	```

### Test scripts

	chmod +x /etc/zabbix/scripts/nginx.sh

	# /etc/zabbix/scripts/nginx.sh
	Usage /etc/zabbix/scripts/nginx.sh {active|accepts|handled|requests|reading|writing|waiting}
	# /etc/zabbix/scripts/nginx.sh accepts
	8287

	# systemctl restart zabbix-agent.service

### Test Agent

	# yum install -y zabbix-get

	# zabbix_get -s <agent_ip_address> -k 'nginx.status[accepts]'
	109

Import Template
-----
	Import file: choice xml file
	click "import" button

	Imported successfully
