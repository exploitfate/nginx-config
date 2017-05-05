Choose latest package at `https://nginx.org/packages/ubuntu/pool/nginx/n/nginx-nr-agent/`

`wget https://nginx.org/packages/ubuntu/pool/nginx/n/nginx-nr-agent/nginx-nr-agent_2.0.0-9_all.deb`

`sudo apt install docutils-common docutils-doc python-daemon python-docutils python-lockfile python-pil python-pygments python-roman python-setproctitle`
`sudo dpkg -i nginx-nr-agent_2.0.0-9_all.deb`


Replace `<ip_address>` below with `127.0.0.1` or Amazon Elastic IP Address when Amazon EC2

`sudo nano /etc/nginx/sites-available/status`

```
server {
	server_name <ip_address>;
	listen 80;

	root /var/www/html;

	location / {
		try_files $uri $uri/ =404;
	}

	location /status {
		stub_status on;
		access_log   off;
		allow <ip_address>;
		deny all;
	}
}
```

`sudo ln -s /etc/nginx/sites-available/status /etc/nginx/sites-enabled/`

`sudo nginx -t`

`sudo service nginx restart`

`sudo nano /etc/nginx-nr-agent/nginx-nr-agent.ini`
```
...
newrelic_license_key=YOUR_LICENSE_KEY_HERE
...
[source1]
name=appname
url=http://<ip_address>/status
...
```

Run monitor 

Ubuntu 14.04
`sudo service nginx-nr-agent start`

Ubuntu 16.04
`sudo /usr/bin/nginx-nr-agent.py start`
