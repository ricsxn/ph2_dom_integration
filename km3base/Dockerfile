#
# Base image for KM3 Integration test container
#
# Build with: docker build -t km3/km3base:1.0 .
#
FROM centos:centos7.9.2009 

ENV TESTUSR=km3net
ENV TESTPSS=pyrosoma

# remove some files to enable systemd
#RUN (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == \
#systemd-tmpfiles-setup.service ] || rm -f $i; done); \
#rm -f /lib/systemd/system/multi-user.target.wants/*;\
#rm -f /etc/systemd/system/*.wants/*;\
#rm -f /lib/systemd/system/local-fs.target.wants/*; \
#rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
#rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
#rm -f /lib/systemd/system/basic.target.wants/*;\
#rm -f /lib/systemd/system/anaconda.target.wants/*;
#VOLUME [ "/sys/fs/cgroup" ]

RUN yum clean all &&\
    yum -y update &&\
    yum -y install epel-release &&\
    yum -y groupinstall "Development tools" &&\
    yum -y install dhcp\
                   mlocate\
                   htop\
                   java-11-openjdk\
                   gnome-terminal\
                   net-tools\
                   zsh\
                   cmake\
                   mysql\
                   mariadb-server\
                   mariadb-devel\
                   octave\
                   openssh-server\
                   openssh-clients\
                   initscripts\
                   mariadb-server\
                   git\
                   cmake3\
                   gcc-c++\
                   gcc\
                   binutils\
                   libX11-devel\
                   libXpm-devel\
                   libXft-devel\
                   libXext-devel\
                   openssl-devel\
                   wget\
                   python3\
                   python3-devel\
                   gcc-gfortran\
                   bison\
                   flex\
                   flex-devel\
                   boost-devel\
                   fftw3\
                   fftw3-devel\
                   ncurses\
                   ncurses-devel\                   
                   sudo\
                   libzstd\
                   npm\
                   evince &&\
    echo "$TESTUSR    ALL=NOPASSWD: ALL" >> /etc/sudoers                   

RUN pip3 install -U pip setuptools

RUN adduser $TESTUSR
RUN echo root:$TESTPSS | chpasswd
RUN echo $TESTUSR:$TESTPSS | chpasswd

USER $TESTUSR
WORKDIR /home/$TESTUSR
