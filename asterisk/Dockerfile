FROM centos:centos7
MAINTAINER e.obulkasim@students.ephec.be

RUN yum update -y
RUN yum install -y epel-release
RUN yum install -y git kernel-headers gcc gcc-c++ cpp \
    ncurses ncurses-devel libxml2 libxml2-devel sqlite \
    sqlite-devel openssl-devel newt-devel kernel-devel \
    libuuid-devel net-snmp-devel xinetd tar jansson-devel \
    make bzip2 gettext

WORKDIR /tmp
# pjproject is needed to run asterisk 13
RUN git clone -b pjproject-2.4.5 --depth 1 https://github.com/asterisk/pjproject.git

# Build the project
WORKDIR /tmp/pjproject
RUN ./configure \
    --prefix=/usr \
    --libdir=/usr/lib64 \
    --enable-shared \
    --disable-sound \
    --disable-resample \
    --disable-video \
    --disable-opencore-amr 1> /dev/null

RUN make dep 1> /dev/null
RUN make 1> /dev/null
RUN make install
# Reload cnfig
RUN ldconfig
RUN ldconfig -p | grep pj

ENV AUTOBUILD_UNIXTIME 123124
WORKDIR /tmp

# Download asterisk.
RUN git clone -b certified/13.8 --depth 1 https://gerrit.asterisk.org/asterisk
WORKDIR /tmp/asterisk

# Configure
RUN ./configure --libdir=/usr/lib64 1> /dev/null
# Remove the native build option
# from: https://wiki.asterisk.org/wiki/display/AST/Building+and+Installing+Asterisk
RUN make menuselect.makeopts
RUN menuselect/menuselect \
  --disable BUILD_NATIVE \
  --enable cdr_csv \
  --enable chan_sip \
  --enable res_snmp \
  --enable res_http_websocket \
  --enable res_hep_pjsip \
  --enable res_hep_rtcp \
  menuselect.makeopts

# Continue with a standard make.
RUN make 1> /dev/null
RUN make install 1> /dev/null
RUN make samples 1> /dev/null
WORKDIR /

# Update max number of open files.
RUN sed -i -e 's/# MAXFILES=/MAXFILES=/' /usr/sbin/safe_asterisk

# Copy config files to appropriate directory
ADD users.conf /etc/asterisk/users.conf
ADD sip.conf /etc/asterisk/sip.conf
ADD extensions.conf /etc/asterisk/extensions.conf
ADD voicemail.conf /etc/asterisk/voicemail.conf

EXPOSE 5060/udp
EXPOSE 5060/tcp
EXPOSE 9080
EXPOSE 11101/udp
EXPOSE 11102/udp
EXPOSE 11103/udp
EXPOSE 11104/udp
EXPOSE 11105/udp
EXPOSE 11106/udp
EXPOSE 11107/udp
EXPOSE 11108/udp
EXPOSE 11109/udp
EXPOSE 11110/udp
EXPOSE 11111/udp


CMD asterisk -f
