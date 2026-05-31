FROM rockylinux:9

RUN dnf install -y net-snmp net-snmp-utils && \
    dnf clean all

COPY conf/snmpd.conf /etc/snmp/snmpd.conf

EXPOSE 161/udp

CMD ["snmpd", "-f", "-Lo"]
