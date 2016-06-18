Ansible role - Nginx server
=====
[![Build Status](https://travis-ci.org/repleo/ansible-role-nginx.svg?branch=master)](https://travis-ci.org/repleo/ansible-role-nginx)
[![Ansible Galaxy](http://img.shields.io/badge/galaxy-repleo.nginx-660198.svg?style=flat)](https://galaxy.ansible.com/repleo/nginx)

This role installs and configures the nginx web server. The user can specify
any http configuration parameters they wish to apply their site. Any number of
sites can be added with configurations of your choice.

Requirements
------------

This role requires Ansible 1.4 or higher and platform requirements are listed
in the metadata file.

Role Variables
--------------

The variables that can be passed to this role and a brief description about
them are as follows.

    # Force creation of nginx.conf. Normally, nginx.conf is only written if it does not exist.
    # If this parameter is true, it will override the current nginx.conf
    create_nginx_conf: true 

    # The max clients allowed
    nginx_max_clients: 512                                

    # A hash of the http paramters. Note that any
    # valid nginx http paramters can be added here.
    # (see the nginx documentation for details.)
    nginx_http_params:                                    
      sendfile: "on"                                      
      tcp_nopush: "on"
      tcp_nodelay: "on"
      keepalive_timeout: "65"
      access_log: "/var/log/nginx/access.log"
      error_log: "/var/log/nginx/error.log"

    # A list of hashs that define the servers for nginx,
    # as with http parameters. Any valid server parameters
    # can be defined here.
    nginx_sites:
    - file_name: foo
      listen: 8080
      server_name: localhost
      root: "/tmp/site1"
      ssl:
        local_keystore_dir: "{{ playbook_dir }}/files/"
        key: "ssl.key"
        certificate: "ssl_chain.pem"
      locations: 
        - name: /
          lines:
            - "try_files: $uri $uri/ /index.html"
        - name: /images/
          lines:
            - "try_files: $uri $uri/ /index.html"
    - file_name: bar
      listen: 9090
      server_name: ansible
      root: "/tmp/site2"
      locations: 
        - name: /
          lines:
            - "try_files: $uri $uri/ /index.html"
        - name: /images/
          lines:
            - "try_files: $uri $uri/ /index.html"
               

Examples
========

1) Install nginx with HTTP directives of choices, but with no sites
configured:

    - hosts: all
      roles:
      - {role: nginx,
            create_nginx_conf: true,
            nginx_http_params: { sendfile: "on",
                                               access_log: "/var/log/nginx/access.log"},
            nginx_sites: [] 
          }


2) Install nginx with different HTTP directives than previous example, but no
sites configured.

    - hosts: all
      roles:
      - {role: nginx,
            create_nginx_conf: true,
            nginx_http_params: { tcp_nodelay: "on",
                                               error_log: "/var/log/nginx/error.log"}, 
            nginx_sites: []
          }

Note: Please make sure the HTTP directives passed are valid, as this role
won't check for the validity of the directives. See the nginx documentation
for details.

3) Install nginx and add a site to the configuration.

    - hosts: all

      roles:
      - { role: nginx,
          create_nginx_conf: true,
          nginx_http_params: { tcp_nodelay: "on",
                                           error_log: "/var/log/nginx/error.log"},
          nginx_sites: [
            - file_name: bar,
              server_name: localhost,
              listen: 8080,
              locations: [
                - name: /,
                  lines: [
                    - "try_files: $uri $uri/ /index.html"
		     ]
                - name: /images/,
                  lines: [
                    - "try_files: $uri $uri/ /index.html"
		     ]
		   ]
		 ] }            
 
Note: Each site added is represented by list of hashes, and the configurations
generated are populated in `/etc/nginx/sites-available/` and have corresponding
symlinks from `/etc/nginx/sites-enabled/`

The file name for the specific site configuration is specified in the hash
with the key "file_name", any valid server directives can be added to hash.


4) Install Nginx and add 2 sites (different method)

    - hosts: all

      roles:
      - { role: nginx,
          create_nginx_conf: true,
          nginx_http_params: { tcp_nodelay: "on",
                                           error_log: "/var/log/nginx/error.log"},
          nginx_sites: [
            - file_name: bar,
              server_name: localhost,
              listen: 8080,
              locations: [
                - name: /,
                  lines: [
                    - "try_files: $uri $uri/ /index.html"
		     ]
                - name: /images/,
                  lines: [
                    - "try_files: $uri $uri/ /index.html"
		     ]
		 ] }
            - file_name: bar,
              server_name: ansible,
              listen: 9090,
              root: "/tmp/site2",
              locations: [
                - name: /,
                  lines: [
                    - "try_files: $uri $uri/ /index.html"
		     ]
                - name: /images/,
                  lines: [
                    - "try_files: $uri $uri/ /index.html"
		     ]
		 ] }  


5) Add virtual hosts to existing Nginx install (and install optionally Nginx if not installed yet)

    - hosts: all

      roles:
      - { role: nginx,
          nginx_sites: [
            - file_name: bar,
              server_name: localhost,
              listen: 8080,
              locations: [
                - name: /,
                  lines: [
                    - "try_files: $uri $uri/ /index.html"
		     ]
                - name: /images/,
                  lines: [
                    - "try_files: $uri $uri/ /index.html"
		     ]
		 ] }
            - file_name: bar,
              server_name: ansible,
              listen: 9090,
              root: "/tmp/site2",
              locations: [
                - name: /,
                  lines: [
                    - "try_files: $uri $uri/ /index.html"
		     ]
                - name: /images/,
                  lines: [
                    - "try_files: $uri $uri/ /index.html"
		     ]
		  ]
		] }  

Note: without the parameter create_nginx_conf: true the role will not overwrite nginx.conf. This option allows to install virtual hosts to an existing nginx installation based on this role, i.e. installation scripts for different services using this role.

6) Example of an HTTPS server including installation of keys

    - hosts: all

      roles:
      - { role: nginx,
          nginx_sites: [
            - file_name: bar,
              server_name: localhost,
              listen: 8080,
              ssl: {
                  local_keystore_dir: "{{ playbook_dir }}/files/",
                  key: localhost.key,
                  certificate: localhost_chain.pem
              },
              locations: [
                - name: /,
                  lines: [
                    - "try_files: $uri $uri/ /index.html"
		     ]
                - name: /images/,
                  lines: [
                    - "try_files: $uri $uri/ /index.html"
		     ]
		  ] 
		] }  

Note: the ssl key and cert should be available in the calling project in the directory files.

Handlers
--------
The NGINX role provides two handlers:
- reload nginx
- restart nginx

Reloading the nginx configuration offers you the ability to update your webserver without downtime. However, it might occur old processes are not updated. Restart will ensure that the webserver is killed and restarted again and will cause (a short) downtime.

Example:

	- name: template configuration file
	  template: src=template.j2 dest=/etc/foo.conf
	  notify:
	     - restart nginx

Dependencies
------------

None

License
-------

BSD

Author Information
------------------

Repleo, Amstelveen, Holland -- www.repleo.nl  
Jeroen Arnoldus (jeroen@repleo.nl)

Original version by:

Benno Joy


