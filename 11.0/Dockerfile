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


#Install Arg LOC.
RUN set -x; \
        wget https://github.com/ingadhoc/odoo-argentina/archive/11.0.zip -O /mnt/extra-addons/ingadhoc-odoo-argentina.zip \
        && wget https://github.com/ingadhoc/account-financial-tools/archive/11.0.zip -O /mnt/extra-addons/ingadhoc-account-financial-tools.zip \
        && wget https://github.com/ingadhoc/account-payment/archive/11.0.zip -O /mnt/extra-addons/ingadhoc-account-payment.zip \
        && wget https://github.com/ingadhoc/aeroo_reports/archive/11.0.zip -O /mnt/extra-addons/ingadhoc-aeroo_reports.zip \
        && wget https://github.com/ingadhoc/miscellaneous/archive/11.0.zip -O /mnt/extra-addons/ingadhoc-miscellaneous.zip \
        && wget https://github.com/ingadhoc/argentina-reporting/archive/11.0.zip -O /mnt/extra-addons/ingadhoc-argentina-reporting.zip \
        && wget https://github.com/ingadhoc/reporting-engine/archive/11.0.zip -O /mnt/extra-addons/ingadhoc-reporting-engine.zip \
        && wget https://github.com/ingadhoc/argentina-sale/archive/11.0.zip -O /mnt/extra-addons/ingadhoc-argentina-sale.zip \
        && wget https://github.com/ingadhoc/stock/archive/11.0.zip -O /mnt/extra-addons/ingadhoc-stock.zip \
        && wget https://apps.odoo.com/loempia/download/partner_identification/11.0.1.0.0/4gXzqj52wzlBc64roBAw7v.zip -O /mnt/extra-addons/partner_identification.zip \
        && unzip /mnt/extra-addons/partner_identification.zip \
        && unzip /mnt/extra-addons/ingadhoc-odoo-argentina.zip \
        && unzip /mnt/extra-addons/ingadhoc-account-financial-tools.zip \
        && unzip /mnt/extra-addons/ingadhoc-account-payment.zip \
        && unzip /mnt/extra-addons/ingadhoc-aeroo_reports.zip \
        && unzip /mnt/extra-addons/ingadhoc-miscellaneous.zip \
        && unzip /mnt/extra-addons/ingadhoc-argentina-reporting.zip \
        && unzip /mnt/extra-addons/ingadhoc-reporting-engine.zip \
        && unzip /mnt/extra-addons/ingadhoc-argentina-sale.zip \
        && unzip /mnt/extra-addons/ingadhoc-stock.zip \
        && rm /mnt/extra-addons/*.zip \
        && pip3 install wheel \
        && chown -R odoo:odoo /mnt/extra-addons/ \
        && chmod -R 775 /mnt/extra-addons/ \
        && pip3 install -r /mnt/extra-addons/odoo-argentina-11/requirements.txt \
        && pip3 install -r /mnt/extra-addons/aeroo_reports/requirements.txt \
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
        


VOLUME ["/var/lib/odoo", "/mnt/extra-addons"]
# Expose Odoo services
EXPOSE 8069 8071

# Set the default config file
ENV ODOO_RC /etc/odoo/odoo.conf

# Set default user when running the container
USER odoo

ENTRYPOINT ["/entrypoint.sh"]
CMD ["odoo"]
