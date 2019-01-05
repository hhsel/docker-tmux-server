FROM arm64v8/debian:stretch-slim AS builder

RUN apt-get update
# base packages
RUN apt-get install -y build-essential curl
# dependency for openssh
RUN apt-get install -y autoconf libz-dev
# dependency for tmux
RUN apt-get install -y libevent-dev ncurses-dev

WORKDIR /tmp
RUN curl -L https://github.com/openssl/openssl/archive/OpenSSL_1_1_1a.tar.gz > openssl.tar.gz
RUN tar zxvf openssl.tar.gz
WORKDIR /tmp/openssl-OpenSSL_1_1_1a
RUN ./config --prefix=/opt/openssl
RUN make
RUN make install_sw

WORKDIR /tmp
RUN useradd sshd
RUN useradd tmux
RUN curl -L https://github.com/openssh/openssh-portable/archive/V_7_9_P1.tar.gz > openssh.tar.gz
RUN tar zxvf openssh.tar.gz
WORKDIR /tmp/openssh-portable-V_7_9_P1
RUN autoreconf
RUN LD_LIBRARY_PATH=/opt/openssl/lib ./configure --prefix=/opt/openssh --with-ssl-dir=/opt/openssl
RUN make
RUN LD_LIBRARY_PATH=/opt/openssl/lib make install
RUN mkdir -p /home/tmux/.ssh
RUN chown -R tmux:tmux /home/tmux
RUN echo '/opt/openssl/lib' > /etc/ld.so.conf.d/00-openssl.conf
RUN ldconfig

WORKDIR /tmp
RUN curl -L https://github.com/tmux/tmux/releases/download/2.8/tmux-2.8.tar.gz > tmux.tar.gz
RUN tar zxvf tmux.tar.gz
WORKDIR /tmp/tmux-2.8
RUN ./configure --prefix=/opt/tmux
RUN make
RUN make install

# stage 2, merging only built apps to its base and install only needed dependencies to run them
FROM arm64v8/debian:stretch-slim

COPY --from=builder /opt /opt
# these dependencies cannot be installed incrementally, to minimize image size, so we need to install in a traditional way
RUN apt-get update && apt-get install -y \
    rsyslog supervisor libevent-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN useradd sshd && useradd tmux
RUN mkdir -p /home/tmux/.ssh && chown -R tmux:tmux /home/tmux
RUN echo 'tmux:dummypasswordforlogin' | chpasswd
RUN chsh -s /bin/bash tmux

RUN echo '/opt/openssl/lib' > /etc/ld.so.conf.d/00-openssl.conf && ldconfig
RUN mkdir /var/empty
RUN ln -s /opt/openssl/bin/openssl /usr/bin/openssl && \
    ln -s /opt/openssh/sbin/sshd /usr/sbin/sshd && \
    ln -s /opt/openssh/bin/ssh /usr/bin/ssh && \
    ln -s /opt/tmux/bin/tmux /usr/bin/tmux

RUN ln -sf /dev/stdout /var/log/auth.log
RUN chmod -R 700 /home/tmux/.ssh

COPY files/supervisord.conf /etc/supervisord.conf
COPY files/sshd_config /opt/openssh/etc/sshd_config
COPY files/rsyslog.conf /etc/rsyslog.conf

EXPOSE 22
ENTRYPOINT ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]
