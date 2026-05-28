# Use the Ubuntu 26:04 base image
FROM ubuntu:24.04

# Avoid prompts during package installation
ENV TZ=US \
    DEBIAN_FRONTEND=noninteractive

# Update packages and install all required packages
RUN apt update && apt upgrade --assume-yes && \
    apt install -y openssh-server sudo git wget python3-venv build-essential libproj-dev \
                       python3-pip libcairo2-dev cmake libgl1 openbox obconf \
                       libegl1 python3-tk tigervnc-standalone-server tigervnc-common \
                       libxcb-cursor-dev libqt6core6 libqt6gui6 libqt6widgets6\
                       bzip2 libreadline6-dev openssl libssl-dev  zlib1g-dev  libbz2-dev \
                       libreadline-dev libsqlite3-dev libffi-dev liblzma-dev tk-dev xz-utils \
                       libglu1-mesa mesa-utils curl pyqt5-dev-tools 

     

# Create the SSH directory and configure permissions
RUN mkdir /var/run/sshd && chmod 0755 /var/run/sshd

# Add a new user 'dockeruser' and set a password
RUN useradd -m -s /bin/bash dockeruser

# Optional: Add the user to the sudoers to allow administrative actions
RUN echo 'dockeruser ALL=(ALL) NOPASSWD: ALL' > /etc/sudoers.d/dockeruser && \
    chmod 0440 /etc/sudoers.d/dockeruser

# Enable password authentication in the SSH configuration
RUN sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config

# Optional: Disable root login via SSH
RUN echo "PermitRootLogin no" >> /etc/ssh/sshd_config

USER dockeruser

RUN curl https://pyenv.run |  bash
RUN ls  -al /home/dockeruser
RUN  export PYENV_ROOT="$HOME/.pyenv" && \
     command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH" && \
     eval "$(pyenv init -)" && \
     echo "install..." && \
     pyenv install 3.10.11 && \
     ls -al ~/.pyenv && \
     cd && \
     echo "local..." && \
     pyenv local 3.10.11 && \ 
     echo "global..." && \
     pyenv global 3.10.11

RUN cd && curl -sSL https://install.python-poetry.org | python3 - && \
    export PATH="/home/dockeruser/.local/bin:$PATH" && \
    git clone https://github.com/mlapaj/FlatCAM_2024_ml.git && \
    mv FlatCAM_2024_ml FlatCAM && \
    cd FlatCAM/FlatCAM && \
    touch README.md && \
    poetry install --no-root && \
    pyrcc5 resources_dark.qrc -o resources_dark.py && \
    pyrcc5 resources.qrc -o resources.py

# Configure VNC password
RUN mkdir -p ~/.vnc

USER root
RUN echo '#!/bin/bash\n\
unset SESSION_MANAGER\n\
unset DBUS_SESSION_BUS_ADDRESS\n\
openbox &\n\
export PATH="/home/dockeruser/.local/bin:$PATH" && \
cd /home/dockeruser/FlatCAM/FlatCAM &&\n\
while true; do poetry run python FlatCAM.py;done' > /home/dockeruser/.vnc/xstartup && \
chmod +x /home/dockeruser/.vnc/xstartup

# Startup script
RUN echo '#!/bin/bash\n\
su - dockeruser -c "echo \"$FLATCAM_VNC_PASS\" | vncpasswd -f > ~/.vnc/passwd"\n\
su - dockeruser -c "chmod 600  ~/.vnc/passwd"\n\
echo "dockeruser:$FLATCAM_SSH_PASS" | chpasswd\n\
service ssh start\n\
su - dockeruser -c "vncserver :1 -geometry 1280x720 -depth 24 -localhost no"\n\
tail -f /dev/null' > /start.sh && \
chmod +x /start.sh

RUN mkdir /data && chown dockeruser /data

# Expose the SSH+VNC port
EXPOSE 22 5901

# Run the SSH server    
CMD ["/start.sh"]
