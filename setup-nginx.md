# Nginx configurations

## Enable and disable sites via script

create script and make it executable
```
$ sudo nano /usr/bin/nginx_modsite
```

option 1:
<details>

```bash
#!/bin/bash

##
#  File:
#    nginx_modsite
#  Description:
#    Provides a basic script to automate enabling and disabling websites found
#    in the default configuration directories:
#      /etc/nginx/sites-available and /etc/nginx/sites-enabled
#    For easy access to this script, copy it into the directory:
#      /usr/local/sbin
#    Run this script without any arguments or with -h or --help to see a basic
#    help dialog displaying all options.
##

# Copyright (C) 2010 Michael Lustfield <mtecknology@ubuntu.com>

# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
# 1. Redistributions of source code must retain the above copyright
#    notice, this list of conditions and the following disclaimer.
# 2. Redistributions in binary form must reproduce the above copyright
#    notice, this list of conditions and the following disclaimer in the
#    documentation and/or other materials provided with the distribution.
#
# THIS SOFTWARE IS PROVIDED BY AUTHOR AND CONTRIBUTORS ``AS IS'' AND
# ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
# ARE DISCLAIMED.  IN NO EVENT SHALL AUTHOR OR CONTRIBUTORS BE LIABLE
# FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
# DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS
# OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION)
# HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT
# LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY
# OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF
# SUCH DAMAGE.

##
# Default Settings
##

NGINX_CONF_FILE="$(awk -F= -v RS=' ' '/conf-path/ {print $2}' <<< $(nginx -V 2>&1))"
NGINX_CONF_DIR="${NGINX_CONF_FILE%/*}"
NGINX_SITES_AVAILABLE="$NGINX_CONF_DIR/sites-available"
NGINX_SITES_ENABLED="$NGINX_CONF_DIR/sites-enabled"
SELECTED_SITE="$2"

##
# Script Functions
##

ngx_enable_site() {
    [[ ! "$SELECTED_SITE" ]] &&
        ngx_select_site "not_enabled"

    [[ ! -e "$NGINX_SITES_AVAILABLE/$SELECTED_SITE" ]] && 
        ngx_error "Site does not appear to exist."
    [[ -e "$NGINX_SITES_ENABLED/$SELECTED_SITE" ]] &&
        ngx_error "Site appears to already be enabled"

    ln -sf "$NGINX_SITES_AVAILABLE/$SELECTED_SITE" -T "$NGINX_SITES_ENABLED/$SELECTED_SITE"
    ngx_reload
}

ngx_disable_site() {
    [[ ! "$SELECTED_SITE" ]] &&
        ngx_select_site "is_enabled"

    [[ ! -e "$NGINX_SITES_AVAILABLE/$SELECTED_SITE" ]] &&
        ngx_error "Site does not appear to be \'available\'. - Not Removing"
    [[ ! -e "$NGINX_SITES_ENABLED/$SELECTED_SITE" ]] &&
        ngx_error "Site does not appear to be enabled."

    rm -f "$NGINX_SITES_ENABLED/$SELECTED_SITE"
    ngx_reload
}

ngx_list_site() {
    echo "Available sites:"
    ngx_sites "available"
    echo "Enabled sites:"
    ngx_sites "enabled"
}

##
# Helper Functions
##

ngx_select_site() {
    sites_avail=($NGINX_SITES_AVAILABLE/*)
    sa="${sites_avail[@]##*/}"
    sites_en=($NGINX_SITES_ENABLED/*)
    se="${sites_en[@]##*/}"

    case "$1" in
        not_enabled) sites=$(comm -13 <(printf "%s\n" $se) <(printf "%s\n" $sa));;
        is_enabled) sites=$(comm -12 <(printf "%s\n" $se) <(printf "%s\n" $sa));;
    esac

    ngx_prompt "$sites"
}

ngx_prompt() {
    sites=($1)
    i=0

    echo "SELECT A WEBSITE:"
    for site in ${sites[@]}; do
        echo -e "$i:\t${sites[$i]}"
        ((i++))
    done

    read -p "Enter number for website: " i
    SELECTED_SITE="${sites[$i]}"
}

ngx_sites() {
    case "$1" in
        available) dir="$NGINX_SITES_AVAILABLE";;
        enabled) dir="$NGINX_SITES_ENABLED";;
    esac

    for file in $dir/*; do
        echo -e "\t${file#*$dir/}"
    done
}

ngx_reload() {
    read -p "Would you like to reload the Nginx configuration now? (Y/n) " reload
    [[ "$reload" != "n" && "$reload" != "N" ]] && invoke-rc.d nginx reload
}

ngx_error() {
    echo -e "${0##*/}: ERROR: $1"
    [[ "$2" ]] && ngx_help
    exit 1
}

ngx_help() {
    echo "Usage: ${0##*/} [options]"
    echo "Options:"
    echo -e "\t<-e|--enable> <site>\tEnable site"
    echo -e "\t<-d|--disable> <site>\tDisable site"
    echo -e "\t<-l|--list>\t\tList sites"
    echo -e "\t<-h|--help>\t\tDisplay help"
    echo -e "\n\tIf <site> is left out a selection of options will be presented."
    echo -e "\tIt is assumed you are using the default sites-enabled and"
    echo -e "\tsites-disabled located at $NGINX_CONF_DIR."
}

##
# Core Piece
##

case "$1" in
    -e|--enable)    ngx_enable_site;;
    -d|--disable)   ngx_disable_site;;
    -l|--list)  ngx_list_site;;
    -h|--help)  ngx_help;;
    *)      ngx_error "No Options Selected" 1; ngx_help;;
esac
```
</details>

make it executable:
```
$ sudo chmod +x /usr/bin/nginx_modsite
```

#### nginx_modsite commands:
+ list all sites:
```
$ sudo nginx_modsite -l
```
+ enable site "test_website"
```
$ sudo nginx_modsite -e test_website
```
+disable site "test_website"
```
$ sudo nginx_modsite -d test_website
```

option 2:
<details>

```
$ sudo nano /usr/bin/nginx_enable_site
```
and copy this:
```bash
#!/bin/sh
read -p "Website to enable: " site;
ln -s /etc/nginx/sites-available/"$site" /etc/nginx/sites-enabled/
echo "$site enabled. Now run 'service nginx reload'"
```

```
$ sudo nano /usr/bin/nginx_disable_site
```
and copy this:
```bash
#!/bin/sh
read -p "Website to disable: " site;
rm /etc/nginx/sites-enabled/"$site"
echo "$site disabled. Now run 'service nginx reload'"
```

make the scripts executable:
```
$ sudo chmod +x /usr/bin/nginx_enable_site /usr/bin/nginx_disable_site
```
</details>

## nginx site configuration

#### reverse proxy
create a config file for each subdomain:
```
sudo nano /etc/nginx/sites-available/test.wntzn.de
```

copy and adjust this:
```
server {
    listen 80;

    server_name www.wntzn.com;

    location / {
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header Host $host;
        proxy_pass       http://127.0.0.1:9001;
    }
}

server {
    server_name wntzn.com;
    return 301 $scheme://www.wntzn.com$request_uri;
}
```

### management commands

stop web server
```
$ sudo systemctl stop nginx
```

start web server
```
$ sudo systemctl start nginx
```

restart web server
```
$ sudo systemctl restart nginx
```

reload configuration without dropping connections
```
$ sudo systemctl reload nginx
```

disable auto start when server boots
```
$ sudo systemctl disable nginx
```

enable service to start at boot
```
$ sudo systemctl enable nginx
```

## nginx directories

### content
```
# webcontent
/var/www/html
```

### server configuration
```
# configuration directory
/etc/nginx

# main nginx configuration file
/etc/nginx/nginx.conf

# stores per-site server blocks
/etc/nginx/sites-available/

# stores enabled per-site server blocks
/etc/nginx/sites-enabled/

# configuration fragments (repeatable configuration segments)
/etc/nginx/snippets
```

### server logs
```
# log of every request to the webserver
/var/log/nginx/access.log

# any nginx errors
/var/log/nginx/error.log
```
