FROM ubuntu:24.04
SHELL ["/bin/bash", "-c"]

ARG TZ="UTC"
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && \
    echo $TZ > /etc/timezone


# Set working directory
WORKDIR /usr/src/app

# Install necessary system dependencies
RUN apt-get update && \
    apt-get -y install python3 python3-pip python3-venv git curl golang-go


# Clone Caldera repository
RUN git clone --recursive --branch 5.0.0 https://github.com/mitre/caldera.git /usr/src/app

WORKDIR /usr/src/app

# Ensure Stockpile plugin exists (this step is optional if you know it's present)
RUN ls plugins/stockpile || (echo "Stockpile plugin missing! Something went wrong." && exit 1)

# Set up python virtualenv
ENV VIRTUAL_ENV=/opt/venv/caldera
RUN python3 -m venv $VIRTUAL_ENV
ENV PATH="$VIRTUAL_ENV/bin:$PATH"

# Install pip requirements
RUN pip install setuptools
RUN pip install --no-cache-dir -r requirements.txt
RUN pip install docker

# Clone the Modbus Plugin (OT plugin)
RUN git clone https://github.com/mitre/modbus.git plugins/modbus

# Ensure config file exists & add Modbus plugin configuration
RUN python3 -c "import app; import app.utility.config_generator; app.utility.config_generator.ensure_local_config();" && \
    echo -e 'plugins:\n  - modbus' >> conf/local.yml && \
    echo -e 'crypt_salt: randomsalt' >> conf/local.yml && \
    echo -e 'encryption_key: myencryptionkey' >> conf/local.yml &&\
    echo -e "host: 0.0.0.0" >> conf/local.yml && \
    echo -e "port: 8888" >> conf/local.yml && \
    sed -i '/\- atomic/d' conf/local.yml;


# Install Go dependencies
WORKDIR /usr/src/app/plugins/sandcat/gocat
RUN go mod tidy && go mod download

WORKDIR /usr/src/app/plugins/sandcat
RUN ./update-agents.sh

RUN mkdir /tmp/gocatextensionstest

RUN cp -R ./gocat /tmp/gocatextensionstest/gocat
RUN cp -R ./gocat-extensions/* /tmp/gocatextensionstest/gocat/

RUN cp ./update-agents.sh /tmp/gocatextensionstest/update-agents.sh

WORKDIR /tmp/gocatextensionstest

RUN mkdir /tmp/gocatextensionstest/payloads

RUN ./update-agents.sh

# Clone atomic red team repo for the atomic plugin
RUN if [ ! -d "/usr/src/app/plugins/atomic/data/atomic-red-team" ]; then   \
    git clone --depth 1 https://github.com/redcanaryco/atomic-red-team.git \
        /usr/src/app/plugins/atomic/data/atomic-red-team;                  \
fi

WORKDIR /usr/src/app/plugins/emu

# If emu is enabled, complete necessary installation steps
RUN if [ $(grep -c "\- emu" ../../conf/local.yml)  ]; then \
    apt-get -y install zlib1g unzip;                \
    pip3 install -r requirements.txt;               \
    ./download_payloads.sh;                         \
fi

WORKDIR /usr/src/app

# (Additional build steps go here...)
# Install Node.js, npm and build VueJS front-end, etc.
# Install Node.js, npm, and other build VueJS front-end
RUN apt-get update && \
    apt-get install -y nodejs npm && \
    # Directly use npm to install dependencies and build the application
    (cd plugins/magma && npm install) && \
    (cd plugins/magma && npm run build) && \
    # Remove Node.js, npm, and other unnecessary packages
    apt-get remove -y nodejs npm && \
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*


WORKDIR /usr/src/app

STOPSIGNAL SIGINT

# Default HTTP port for web interface and agent beacons over HTTP
EXPOSE 8888

# Default HTTPS port for web interface and agent beacons over HTTPS (requires SSL plugin to be enabled)
EXPOSE 8443

# TCP and UDP contact ports
EXPOSE 7010
EXPOSE 7011/udp

# Websocket contact port
EXPOSE 7012

# Default port to listen for DNS requests for DNS tunneling C2 channel
EXPOSE 8853

# Default port to listen for SSH tunneling requests
EXPOSE 8022

# Default FTP port for FTP C2 channel
EXPOSE 2222

ENTRYPOINT ["bash", "-c", "python -X dev server.py "]
