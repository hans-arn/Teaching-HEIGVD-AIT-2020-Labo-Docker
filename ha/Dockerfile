# Base image is the Official HAProxy
FROM haproxy:2.2

LABEL Doran Kayoumi <doran.kayoumi@heig-vd.ch> & Jérôme Arn <jerome.arn@heig-vd.ch>

# Install some tools
RUN apt-get update && apt-get -y install wget curl vim iputils-ping rsyslog xz-utils

# Download and install S6 overlay
RUN curl -sSLo /tmp/s6.tar.gz https://github.com/just-containers/s6-overlay/releases/download/v2.1.0.2/s6-overlay-amd64.tar.gz \
  && tar xzf /tmp/s6.tar.gz -C / \
  && rm -f /tmp/s6.tar.gz

# Install serf (for decentralized cluster membership: https://www.serf.io/)
RUN mkdir -p /opt/bin 
RUN curl -sSLo /tmp/serf.gz https://releases.hashicorp.com/serf/0.8.2/serf_0.8.2_linux_amd64.zip \
    && gunzip -c /tmp/serf.gz > /opt/bin/serf \
    && chmod 755 /opt/bin/serf \
    && rm -f /tmp/serf.gz

# Install NodeJS
RUN curl -sSLo /tmp/node.tar.xz https://nodejs.org/dist/v14.15.1/node-v14.15.1-linux-x64.tar.xz \
  && tar -C /usr/local --strip-components 1 -xf /tmp/node.tar.xz \
  && rm -f /tmp/node.tar.xz

# Install the handlebars-cmd node module and its dependencies
RUN npm install -g handlebars-cmd

# Copy the S6 service and make the run script executable
COPY services/ha /etc/services.d/ha
RUN chmod +x /etc/services.d/ha/run

# Copy the Serf service and make the script executable
COPY services/serf /etc/services.d/serf
RUN chmod +x /etc/services.d/serf/run

# Copy events handler scripts
RUN mkdir -p /serf-handlers
COPY scripts/* /serf-handlers/
RUN chmod +x /serf-handlers/*

# Copy the haproxy and syslog config
COPY config/haproxy.cfg /usr/local/etc/haproxy/haproxy.cfg
COPY config/rsyslogd.cfg /etc/rsyslog.d/49-haproxy.conf

# TODO: [HB] Copy the haproxy configuration template
RUN mkdir -p /config
COPY config/haproxy.cfg.hb /config/haproxy.cfg.hb

# Clean up
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN mkdir /nodes

# Expose the ports for Serf
EXPOSE 7946 7373

# Expose the HA proxy ports
EXPOSE 80 1936

# Define an environment variable
ENV ROLE balancer

# This will start S6 as our main process in our container
ENTRYPOINT ["/init"]
