FROM rockylinux:9

RUN dnf install -y net-snmp net-snmp-utils && \
    dnf clean all

COPY conf/snmpd.conf    /etc/snmp/snmpd.conf
COPY conf/snmptrapd.conf /etc/snmp/snmptrapd.conf

RUN mkdir -p /var/log/snmptrapd

COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

EXPOSE 161/udp 162/udp

ENTRYPOINT ["/entrypoint.sh"]
# test
# test3
# test4
