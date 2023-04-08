#
# Base image for KM3 Integration test container
#
# Build with: docker build -t km3/km3test:1.0 .
# Run with: docker run --name km3test --net host -e DISPLAY=192.168.1.157:0.0 -ti -d  km3/km3test:1.0
#
FROM km3/km3base:1.0

ENV JPPREL=v14.4.2

RUN  wget -q http://sftp.km3net.de/CLB_Builds/clb_rev20210315.tar.gz &&\
     tar xvfz clb_rev20210315.tar.gz &&\
     git clone https://km3net%40km3net.de:pyrosoma@git.km3net.de/vpestel/tedi.git
RUN  ROOTTGZ=https://root.cern/download/root_v6.20.06.Linux-centos7-x86_64-gcc4.8.tar.gz &&\
     wget -q $ROOTTGZ -O $(basename $ROOTTGZ) &&\
     tar -xzvf $(basename $ROOTTGZ)
RUN  source root/bin/thisroot.sh &&\
     echo "#!/bin/bash" > install_JPP.sh &&\
     echo "RELEASE=\$1" >> install_JPP.sh &&\
     echo "mkdir -p \$RELEASE" >> install_JPP.sh &&\
     echo "git clone --branch \$RELEASE --recursive https://km3net%40km3net.de:pyrosoma@git.km3net.de/common/jpp.git \$RELEASE" >> install_JPP.sh &&\
     echo "cd \$RELEASE" >> install_JPP.sh &&\
     echo "git submodule init" >> install_JPP.sh &&\
     echo "git submodule update" >> install_JPP.sh &&\
     echo "source setenv.sh" >> install_JPP.sh &&\
     echo "make -j 4" >> install_JPP.sh &&\
     echo "make examples" >> install_JPP.sh &&\
     echo "make DAQ" >> install_JPP.sh &&\
     echo "export PATH=$PATH:\$(pwd)/out/Linux/bin" >> install_JPP.sh &&\
     echo "make install" >> install_JPP.sh &&\
     chmod +x install_JPP.sh &&\
     ./install_JPP.sh $JPPREL
     
RUN  pip3 install km3compass --user &&\
     echo "#!/bin/bash" > $HOME/tedi/setup/env_site.sh &&\
     echo "" >> $HOME/tedi/setup/env_site.sh &&\
     echo "# Adapted for testing site" >> $HOME/tedi/setup/env_site.sh &&\
     echo "#------------------------------------------------------------------------------" >> $HOME/tedi/setup/env_site.sh &&\
     echo "" >> $HOME/tedi/setup/env_site.sh &&\
     echo "#    Directory where the scripts are (where the git repo is cloned)" >> $HOME/tedi/setup/env_site.sh &&\
     echo "export SCRIPTS_DIR='/home/$TESTUSR/tedi/'" >> $HOME/tedi/setup/env_site.sh &&\
     echo "" >> $HOME/tedi/setup/env_site.sh &&\
     echo "#    Directory where CLB dependancies are" >> $HOME/tedi/setup/env_site.sh &&\
     echo "export REMOTE_DIR=\"/home/$TESTUSR/remote/\"" >> $HOME/tedi/setup/env_site.sh &&\
     echo "" >> $HOME/tedi/setup/env_site.sh &&\
     echo "#    Directory to store all the data and files" >> $HOME/tedi/setup/env_site.sh &&\
     echo "export DATA_DIR='/home/$TESTUSR/New_DOM_testing_data/'" >> $HOME/tedi/setup/env_site.sh &&\
     echo "" >> $HOME/tedi/setup/env_site.sh &&\
     echo "#------------------------------------------------------------------------------" >> $HOME/tedi/setup/env_site.sh  &&\
     echo "" >> $HOME/tedi/setup/env_site.sh &&\
     echo "source $HOME/tedi/setup/testing_env.sh" >> $HOME/tedi/setup/env_site.sh &&\
     echo "" >> $HOME/tedi/setup/env_site.sh &&\
     echo ". ~/root/bin/thisroot.sh" >> ~/.bashrc  &&\
     echo "cd ~/$JPPREL" >> ~/.bashrc  &&\
     echo ". setenv.sh" >> ~/.bashrc  &&\
     echo "cd - 2>&1 >/dev/null" >> ~/.bashrc &&\
     echo ". /home/$TESTUSR/tedi/setup/env_site.sh " >> ~/.bashrc

# JPP needs DAQ to be compiled again
RUN /bin/bash -c 'source root/bin/thisroot.sh && cd $JPPREL && source setenv.sh && make DAQ && cd ~/tedi/piezo_mini && make'

# TEDI Configuration (env_site $DATA_DIR)
RUN cd tedi/setup &&\
    echo y | ./genDataDirTreeStruct.sh /home/$TESTUSR/New_DOM_testing_data &&\
    cd -

# Select Java 11
RUN echo "1" | sudo alternatives --config java

# TEDI python configuration
RUN cd tedi &&\
    pip install -r requirements.txt &&\
    pip install PyMySQL

# Local DB
RUN echo "SET PASSWORD FOR root@localhost=PASSWORD('');" > dbuser.sql &&\
    echo "CREATE USER '"$TESTUSR"'@'localhost';" >> dbuser.sql &&\
    echo "GRANT SELECT ON *.* TO '"$TESTUSR"'@'localhost';" >> dbuser.sql &&\
    echo "#!/bin/bash" > dbsetup.sh &&\
    echo "sudo mysql_install_db --user=mysql --ldata=/var/lib/mysql" >> dbsetup.sh &&\
    echo "sudo /usr/bin/mysqld_safe --basedir=/usr --datadir='/var/lib/mysql' &" >> dbsetup.sh &&\
    echo "sleep 3 &&" >> dbsetup.sh &&\
    echo "mysql -u root < dbuser.sql" >> dbsetup.sh &&\
    echo "cd /home/$TESTUSR/tedi/setup" >> dbsetup.sh &&\
    echo "mysql -u root < createDB.sql" >> dbsetup.sh &&\
    chmod +x dbsetup.sh &&\
    ./dbsetup.sh

USER root
WORKDIR /usr

# Fake dhcp
RUN mkdir -p /etc/dhcp &&\
    echo "default-lease-time 600;" >/etc/dhcp/dhcpd.conf &&\
    echo "max-lease-time 7200;" >>/etc/dhcp/dhcpd.conf &&\
    echo "ddns-update-style none;" >>/etc/dhcp/dhcpd.conf &&\
    echo "authoritative;" >>/etc/dhcp/dhcpd.conf &&\
    echo "" >>/etc/dhcp/dhcpd.conf &&\
    echo "# keep this line unchanged! it is needed for a proper working new_dom.py script" >>/etc/dhcp/dhcpd.conf &&\
    echo "" >>/etc/dhcp/dhcpd.conf &&\
    echo "subnet 192.168.1.0 netmask 255.255.255.0 {" >>/etc/dhcp/dhcpd.conf &&\
    echo "  range dynamic-bootp 192.168.1.10 192.168.1.30;" >>/etc/dhcp/dhcpd.conf &&\
    echo "  #dynamic-bootp-lease-length 2;" >>/etc/dhcp/dhcpd.conf &&\
    echo "}" >>/etc/dhcp/dhcpd.conf

# Network tools
RUN yum install -y nc\
                   arp-scan\
                   traceroute\
                   tcpdump

# Call updatedb for filesystem search
RUN updatedb

EXPOSE 3306
#CMD ["mysqld_safe",  "--basedir=/usr ", "--datadir='/var/lib/mysql'"]
#CMD ["mysqld_safe"]
 
EXPOSE 2222
RUN ssh-keygen -f /etc/ssh/ssh_host_rsa_key -N '' -t rsa
RUN ssh-keygen -f /etc/ssh/ssh_host_ed25519_key -N '' -t ed25519
RUN ssh-keygen -f /etc/ssh/ssh_host_ecdsa_key -N '' -t ecdsa
#CMD ["/usr/sbin/sshd", "-p 2222", "-D"]


# NTP and missing packages
RUN yum install -y ntp\
                   csh\
                   ImageMagick\
                   python3-tkinter

# NTP configuration
ENV NTP_HOST=192.168.1.1
RUN mv /etc/ntp.conf /etc/ntp.conf_ntp &&\
    echo "driftfile /var/lib/ntp/drift" >/etc/ntp.conf &&\
    echo "restrict 127.0.0.1" >>/etc/ntp.conf &&\
    echo "restrict ::1" >>/etc/ntp.conf &&\
    echo "restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap" >>/etc/ntp.conf &&\
    echo "server $NTP_HOST iburst" >>/etc/ntp.conf 


# Script fixes
RUN sed -i s/"if (p->ifa_addr->sa_family == AF_INET  ||"/"if (p->ifa_addr == NULL) continue;\n\tif (p->ifa_addr->sa_family == AF_INET  ||"/ /home/$TESTUSR/$JPPREL/software/JSystem/JNetwork.hh &&\
    sed -i s/^ssh.*/"JLigier-local.sh \$PORT \$OPTION"/ /home/$TESTUSR/$JPPREL/out/Linux/bin/JLigier.sh &&\
    sed -i s/^echo\ \"#\ Setting\ environment\ variables.*/"[[\ \"\$-\"\ =\ *\"i\"*\ ]]\ \&\&\ echo\ \"# Setting environment variables for JPP software\""/ /home/$TESTUSR/$JPPREL/setenv.sh &&\
    source /home/$TESTUSR/root/bin/thisroot.sh &&\
    echo "#!/bin/bash" >> /home/$TESTUSR/recompile.sh &&\
    echo "cd /home/$TESTUSR/\$JPPREL" >> /home/$TESTUSR/recompile.sh &&\
    echo "source setenv.sh" >> /home/$TESTUSR/recompile.sh &&\
    echo "make" >> /home/$TESTUSR/recompile.sh &&\
    echo "make DAQ" >> /home/$TESTUSR/recompile.sh &&\
    echo "make install" >> /home/$TESTUSR/recompile.sh &&\
    chmod +x /home/$TESTUSR/recompile.sh &&\
    /home/$TESTUSR/recompile.sh &&\
    rm -f /home/$TESTUSR/recompile.sh &&\
    su - $TESTUSR -c "echo "" | ssh-keygen -t rsa -q -N ''" &&\
    su - $TESTUSR -c "cat .ssh/id_rsa.pub >> .ssh/authorized_keys" &&\
    chmod 700 /home/$TESTUSR/.ssh/authorized_keys &&\
    echo "end_of_fixes"

# Execution script
COPY km3test /usr/km3test

CMD [ "./km3test" ]
