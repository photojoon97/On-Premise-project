FROM rockylinux:9

RUN dnf install -y net-snmp net-snmp-utils && \
    dnf clean all

COPY conf/snmptrapd.conf /etc/snmp/snmptrapd.conf

RUN mkdir -p /var/log/snmptrapd

EXPOSE 162/udp

CMD ["snmptrapd", "-f", "-Lo"]
