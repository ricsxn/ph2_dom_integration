#
# Base image for KM3 Integration test container
#
# Build with: docker build -t km3/km3test:1.0 .
# Run with: docker run --name km3test --net host -e DISPLAY=$DISPLAY -ti -d ph2_km3base:1.0
#
FROM ph2_km3base:1.0

ENV JPPREL=v17.3.2
ENV TESTUSR=km3net
ENV DOMNET=192.168.1.0
ENV DOMNETMASK=255.255.255.0
ENV DOMNET_DHCP_FROM=192.168.1.10
ENV DOMNET_DHCP_TO=192.168.1.30
ENV NTP_HOST=192.168.1.1
ENV CLB_BUILD="https://sftp.km3net.de/CLB_Builds/clb-client-v1.2.1.tar.gz"

USER $TESTUSR
WORKDIR /home/$TESTUSR

RUN  wget -q https://sftp.km3net.de/CLB_Builds/clbng-v1.0.0b6m1-b6a13952-fw.tar.gz &&\
     tar xvfz clbng-v1.0.0b6m1-b6a13952-fw.tar.gz &&\
     git clone https://km3net%40km3net.de:pyrosoma@git.km3net.de/vpestel/tedi.git
RUN  ROOTTGZ=https://root.cern/download/root_v6.20.06.Linux-centos7-x86_64-gcc4.8.tar.gz &&\
     wget -q $ROOTTGZ -O $(basename $ROOTTGZ) &&\
     tar -xzvf $(basename $ROOTTGZ)
RUN  echo "#!/bin/bash" > install_JPP.sh &&\
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
     chmod +x install_JPP.sh
RUN  source root/bin/thisroot.sh &&\
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
RUN /bin/bash -c "source root/bin/thisroot.sh && cd $JPPREL && source setenv.sh && make DAQ && cd ~/tedi/piezo_mini && make"

# TEDI Configuration (env_site $DATA_DIR)
RUN cd tedi/setup &&\
    echo y | ./genDataDirTreeStruct.sh /home/$TESTUSR/New_DOM_testing_data &&\
    cd -

# Select Java 11
RUN echo "2" | sudo alternatives --config java

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

# cmdr_events contains useful tools for WWRS DOM testing
RUN su - $TESTUSR -c "git clone https://git.km3net.de/rbruno/cmdr_events.git" &&\
    su - $TESTUSR -c "mkdir -p ~/bin" &&\
    su - $TESTUSR -c "cp cmdr_events/tools/* ~/bin"

# Install CLB_BUILD (cmdr)
RUN su - $TESTUSR -c "wget $CLB_BUILD" &&\
    su - $TESTUSR -c "tar xvf `basename $CLB_BUILD`" &&\
    su - $TESTUSR -c "CLBBUILDTAR=`basename $CLB_BUILD` && CLBBUILDDIR=\${CLBBUILDTAR%.*.*} && echo \"export PATH=\$PATH:~/\$CLBBUILDDIR/bin\" >> ~/.bashrc"

# Fix on takePMTData.py
RUN su - $TESTUSR -c "cp -r $JPPREL/software/JDOMProduction $JPPREL/examples/"

# 06.12.2023 Fix on km3db, upgrade package 0.5.1 -> km3db==0.13.1
RUN su - $TESTUSR -c "pip install -U km3db"

# Add ~/.local/bin to PATH
RUN su - $TESTUSR -c "echo \"export PATH=\$PATH:~/.local/bin\" >> ~/.bashrc"

# Execution script
COPY km3test /usr/km3test

CMD [ "./km3test" ]
