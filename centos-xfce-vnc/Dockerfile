# This Dockerfile is used to build an sakuli image based on Centos

FROM centos:7

MAINTAINER Tobias Schneck "tobias.schneck@consol.de"
ENV REFRESHED_AT 2016-06-01

ENV JAVA_VERSION 8u92

ENV DISPLAY :1
ENV NO_VNC_HOME /sakuli/noVNC
ENV VNC_COL_DEPTH 24
ENV VNC_RESOLUTION 1280x1024
ENV VNC_PW vncpassword

ENV SAKULI_DOWNLOAD_URL https://labs.consol.de/sakuli/install

############### xvnc / xfce installation
RUN yum -y install epel-release && yum -y update && yum -y install sudo && yum clean all -y
RUN yum --enablerepo=epel -y -x gnome-keyring --skip-broken groups install "Xfce"
RUN yum -y groups install "Fonts" && yum clean all -y
RUN yum -y install tigervnc-server wget net-tools nss_wrapper gettext && yum clean all -y
### Install noVNC - HTML5 based VNC viewer
RUN mkdir -p $NO_VNC_HOME/utils/websockify \
    && wget -qO- https://github.com/ConSol/noVNC/archive/consol_1.0.0.tar.gz | tar xz --strip 1 -C $NO_VNC_HOME \
    && wget -qO- https://github.com/kanaka/websockify/archive/v0.7.0.tar.gz | tar xz --strip 1 -C $NO_VNC_HOME/utils/websockify \
    && chmod +x -v $NO_VNC_HOME/utils/*.sh

### Install firefox chrome browser
ADD repos /etc/yum.repos.d/
RUN yum -y install firefox google-chrome-stable \
    && echo "alias google-chrome='/usr/bin/google-chrome --user-data-dir'" >> /root/.bashrc \
    && echo "alias google-chrome='/usr/bin/google-chrome -no-default-browser-check --user-data-dir'" >> /etc/bashrc \
    && yum clean all -y

### Install java and java-plugin
RUN yum -y install $SAKULI_DOWNLOAD_URL/3rd-party/java/jre-$JAVA_VERSION-linux-x64.rpm && yum clean all -y
# creat symbolic link for firefox java plugin
RUN ln -s /usr/java/default/lib/amd64/libnpjp2.so /usr/lib64/mozilla/plugins/

# xvnc server porst, if $DISPLAY=:1 port will be 5901
EXPOSE 5901
# novnc web port
EXPOSE 6901

#RUN groupadd -r sakuli -g 433 && \
#    useradd -u 431 -r -g sakuli -d /sakuli -s /sbin/nologin -c "Sakuli user" sakuli && \
#    chown -R sakuli:sakuli /sakuli && \
#    chmod -R go+rw /sakuli

ENV HOME /sakuli

ADD .vnc /sakuli/.vnc
ADD .config /sakuli/.config
ADD Desktop /sakuli/Desktop
ADD scripts /sakuli/scripts
ADD passwd.template /sakuli/passwd.template

RUN chmod +x /sakuli/scripts/*.sh /sakuli/.vnc/xstartup /etc/xdg/xfce4/xinitrc /sakuli/Desktop/*.desktop 
RUN chgrp -R 0 /sakuli
RUN chmod -R g+rw /sakuli
RUN find /sakuli -type d -exec chmod g+x {} +

RUN /bin/dbus-uuidgen > /etc/machine-id

USER 1984

ENTRYPOINT ["/sakuli/scripts/vnc_startup.sh"]
CMD ["--tail-log"]
