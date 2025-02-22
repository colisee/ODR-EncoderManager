# ODR-EncoderManager
OpenDigitalRadio Encoder Manager is a tools to run and configure ODR Encoder easly with a WebGUI.

# Note about version V5.0.1

**Bug fixes**

  * use yaml.safe_load instead yaml.load to avoid crash on recent python version

# Note about version V5.0.0
**Requirement**

  * ODR-AudioEnc v3.3.0
  * ODR-SourceCompanion v1.0.0
  * ODR-PadEnc v3.0.0

**New Feature / Change**

  * Add odr-padenc raw-slides option
  * Add additional supervisor option by encoder (only via editing config file at this time)
  * Display ODR tools version
  * ODR-padenc socket support
  * Change supervisor stderr logging
  * Communication between odr-audioenc/odr-sourcecompanion use socket instead fifo

**Bug fixes**

  * Solve issue with SNMP request on AVT with last firmware (ClockSource)


# INSTALLATION
Please, run the following instructions as `root`
  * Add user __odr__:
    ```
    useradd \
      --create-home \
      --groups dialout,audio \
      odr
    ```
  * Set the password for user __odr__:
    ```
    passwd odr
    ```
  * Create the following directories for user __odr__:
    ```
    mkdir --parent /home/odr/.config/supervisor
    mkdir --parent /home/odr/.state/log
    ```
  * Clone this git repository:
    ```
    cd /home/odr
    git clone https://github.com/opendigitalradio/odr-encodermanager
    ```
  * Add odr-encodermanager to supervisor:
    ```
    cat << EOF | tee /home/odr/.config/supervisor/odr-encodermanager.conf
    [program:odr-encoderManager]
    command=python3 run.py -c /home/odr/.config/odr/odr-encodermanager.json
    directory=/home/odr/odr-encodermanager/
    autostart=true
    autorestart=true
    priority=10
    user=odr
    group=odr
    redirect_stderr=true
    stdout_logfile=/home/odr/.state/log/odr-encodermanager.log
    EOF
    ```
  * Grant ownership to odr:
    ```
    chown --recursive odr:odr /home/odr
    ```
  * Install requirement (debian):
    ```
    apt update
    apt upgrade
    apt install \
      python3-cherrypy3 \
      python3-jinja2 \
      python3-pysnmp4 \
      python3-serial \
      python3-yaml \
      supervisor
    ```
  * Setup the supervisor internet server:
    ```
    if ! [ $(grep inet_http_server /etc/supervisor/supervisord.conf) ]
    then cat << EOF | tee -a /etc/supervisor/supervisord.conf

    [inet_http_server]
    port = 8900
    username = user
    password = pass
    EOF

    fi
    ```
  * Update files supervisor should monitor:
    ```
    if ! [ $(grep /home/odr/.config /etc/supervisor/supervisord.conf) ]
    then sed \
      -i /etc/supervisor/supervisord.conf \
      -e "s:\(files .*$\):\1 /home/odr/.config/supervisor/*.conf:"
    fi
    ```
  * Restart `supervisor`:
    ```
    systemctl restart supervisor.service
    ```

# CONFIGURATION
  * You can edit global configuration, in particular path in this files :
    * /home/odr/.config/odr/odr-encodermanager.json
    * /home/odr/.config/supervisor/odr-encodermanager.conf
  * If you want to change supervisor XMLRPC login/password, you need to edit these files:
    * /etc/supervisor/supervisord.conf
    * /home/odr/.config/odr/odr-encodermanager.json

# RUN
  * point your web browser to : `http://<ip_address>:8080`
  * Login with user `joe` and password `secret` 

# How to set DLS / DL+ / SLS
**Set DLS / DL+ for all encoder**
To set a text metadata used for DLS, use http GET or POST on the Encoder Manager API from your automation software.
> http://{ip}:8080/api/setDLS?dls={artist}-{title}

As an alternative DLS+ tags are automatically activated if you use artist & title parameters:
> http://{ip}:8080/api/setDLS?artist={artist}&title={title}

Many radio automation software can send this information to Encoder Manager API by using a call of this type (for example)
> http://{ip}:8080/api/setDLS?dls=%%artist%% - %%title%%

%%artist%% - %%title%% should be replaced with the expression expected from your radio automation software.

At each events on your playlist (when a track start) the radio automation software will send via this url the appropriate metadata to Encoder Manager API. It will be reflected on the DAB signal.

**Set DLS / DL+ for specific encoder (from version V4.0.0)**
If you want to update DLS / DL+ for a specific encoder, you need to find the uniq_id on Encoder > Manage page under Information button
> http://{ip}:8080/api/setDLS?dls={artist}-{title}&uniq_id={00000000-0000-0000-0000-000000000000}

**Set SLS for all encoder**
To send slide used for SLS, use http POST on the Encoder Manager API.
> curl -X POST -F 'slide_file=@"live.jpg"' http://{ip}:8080/api/setSLS

**Set SLS for specific encoder**
If you want to update SLS for a specific encoder, you need to find the uniq_id on Encoder > Manage page under Information button
> curl -X POST -F 'uniq_id={00000000-0000-0000-0000-000000000000}' -F 'slide_file=@"{file.jpg}"' http://{ip}:8080/api/setSLS

# ADVANCED
  * To use the reboot api (/api/reboot), you need to allow odr user to run shutdown command by adding the line bellow at the end of /etc/sudoers file :
```
odr     ALL=(ALL) NOPASSWD: /sbin/shutdown
```

