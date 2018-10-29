FROM debian:stretch
MAINTAINER Odoo S.A. <info@odoo.com>

# Generate locale C.UTF-8 for postgres and general locale data
ENV LANG C.UTF-8

# Install some deps, lessc and less-plugin-clean-css, and wkhtmltopdf
RUN set -x; \
        apt-get update \
        && apt-get install -y --no-install-recommends \
            ca-certificates \
            curl \
            node-less \
            python3-pip \
            python3-setuptools \
            python3-renderpm \
            libssl1.0-dev \
            xz-utils \
            python3-watchdog \
            unzip \
            wget \
            git \
            python3-dev \
            build-essential \
            nano \
        && curl -o wkhtmltox.tar.xz -SL https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.4/wkhtmltox-0.12.4_linux-generic-amd64.tar.xz \
        && echo '3f923f425d345940089e44c1466f6408b9619562 wkhtmltox.tar.xz' | sha1sum -c - \
        && tar xvf wkhtmltox.tar.xz \
        && cp wkhtmltox/lib/* /usr/local/lib/ \
        && cp wkhtmltox/bin/* /usr/local/bin/ \
        && cp -r wkhtmltox/share/man/man1 /usr/local/share/man/

# Install Odoo
ENV ODOO_VERSION 11.0
ENV ODOO_RELEASE 20181026
RUN set -x; \
        curl -o odoo.deb -SL http://nightly.odoo.com/${ODOO_VERSION}/nightly/deb/odoo_${ODOO_VERSION}.${ODOO_RELEASE}_all.deb \
        && echo '8368d92d62c2605c261fb628f733011fbfb1b84f odoo.deb' | sha1sum -c - \
        && dpkg --force-depends -i odoo.deb \
        && apt-get update \
        && apt-get -y install -f --no-install-recommends \
        && rm -rf /var/lib/apt/lists/* odoo.deb
        


# Copy entrypoint script and Odoo configuration file
RUN pip3 install num2words xlwt
COPY ./entrypoint.sh /
COPY ./odoo.conf /etc/odoo/
RUN chown odoo /etc/odoo/odoo.conf

# Mount /var/lib/odoo to allow restoring filestore and /mnt/extra-addons for users addons
RUN mkdir -p /mnt/extra-addons \
        && chown -R odoo /mnt/extra-addons
VOLUME ["/var/lib/odoo", "/mnt/extra-addons"]

#Install Arg LOC.
COPY ./* /mnt/extra-addons
RUN set -x; \
        pip3 install wheel \
        && chown -R odoo:odoo /mnt/extra-addons/ \
        && chmod -R 775 /mnt/extra-addons/ \
        && cd /mnt/extra-addons/odoo-argentina-11.0 \
        && pip3 install -r requirements.txt \
        && cd /mnt/extra-addons/aeroo_reports-11.0 \
        && pip3 install -r requirements.txt \
        && apt-get install default-jre -y \
        && apt-get install python3-setuptools python3-uno \
        && apt-get install python3-lxml -y \
        && pip3 install jsonrpc2 daemonize \
        && mkdir /opt/aeroo \
        && cd /opt/aeroo \
        && git clone https://github.com/aeroo/aeroo_docs.git \
        && python3 /opt/aeroo/aeroo_docs/aeroo-docs start -c /etc/aeroo-docs.conf \
        && ln -s /opt/aeroo/aeroo_docs/aeroo-docs /etc/init.d/aeroo-docs \
        && sudo update-rc.d aeroo-docs defaults \
        && service aeroo-docs start \
        && service aeroo-docs restart \
        && apt-get install libreoffice libreoffice-script-provider-python -y \
        && echo '#!/bin/sh' | sudo tee /etc/init.d/office \
        && echo '/usr/bin/libreoffice --headless --accept="socket,host=localhost,port=8100,tcpNoDelay=1;urp;"&' \
        && tee -a /etc/init.d/office \
        && chmod +x /etc/init.d/office \
        && update-rc.d office defaults \
        && /etc/init.d/office \
        && apt install unoconv \
        



# Expose Odoo services
EXPOSE 8069 8071

# Set the default config file
ENV ODOO_RC /etc/odoo/odoo.conf

# Set default user when running the container
USER odoo

ENTRYPOINT ["/entrypoint.sh"]
CMD ["odoo"]
