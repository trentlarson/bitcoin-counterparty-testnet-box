# bitcoin-testnet-box docker image


#FROM ubuntu:12.04
# Need 14.04+ 'cause we need sqlite 3.8.5+ for Counterparty build.
#FROM ubuntu:14.04
# This fixes various problems we hit.  See http://phusion.github.io/baseimage-docker/
FROM phusion/baseimage

# add bitcoind from the official PPA
RUN apt-get update
RUN apt-get install --yes python3
RUN apt-get install --yes python-software-properties
RUN apt-get install --yes software-properties-common
RUN add-apt-repository --yes ppa:bitcoin/bitcoin
RUN apt-get update

# install bitcoind (from PPA) and make
RUN apt-get install --yes bitcoind make

# Counterparty steps #1
RUN apt-get update --yes
RUN apt-get install --yes git-core
RUN apt-get install --yes vim
RUN apt-get install --yes curl







# create a non-root user
RUN adduser --disabled-login --gecos "" tester

# run following commands from user's home directory
WORKDIR /home/tester


# copy the testnet-box files into the image
RUN git clone https://github.com/trentlarson/bitcoin-counterparty-testnet-box.git


# Counterparty steps #2
RUN git clone https://github.com/CounterpartyXCP/counterpartyd_build
RUN perl -p -i -e "s/run_as_user = os.environ/run_as_user = 'tester'#os.environ/" /home/tester/counterpartyd_build/setup.py
RUN cd counterpartyd_build; python3 setup.py --with-bootstrap-db --for-user=tester --with-testnet --noninteractive

# These fail since it can't find test_.py
#RUN counterpartyd tests



# These could be run as the non-root user.
#WORKDIR /home/tester/bitcoin-testnet-box
#RUN export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
#RUN source /usr/local/bin/virtualenvwrapper.sh
#RUN mkvirtualenv counterpartyd
#RUN pip install -r /home/tester/counterpartyd_build/dist/counterpartyd/pip-requirements.txt
#RUN pip install apsw



# make tester user own the bitcoin-testnet-box
RUN chown -R tester:tester /home/tester/bitcoin-counterparty-testnet-box
RUN chown -R tester:tester /home/tester/counterpartyd_build
RUN chown -R tester:tester /home/tester/.config

# use the tester user when running the image
# ... but something's wrong 'cause we get this when we run (before we become tester):
#   bash: /root/.bashrc: Permission denied
USER tester

# Edit the testnet config to point to internal Bitcoin daemon #1
RUN perl -p -i -e "s/backend-rpc-port=.+/backend-rpc-port=19001/g" /home/tester/.config/counterpartyd-testnet/counterpartyd.conf
RUN perl -p -i -e "s/backend-rpc-user=.+/backend-rpc-user=admin1/g" /home/tester/.config/counterpartyd-testnet/counterpartyd.conf
RUN perl -p -i -e "s/backend-rpc-password=.+/backend-rpc-password=123/g" /home/tester/.config/counterpartyd-testnet/counterpartyd.conf

# run remaining commands from inside the testnet-box directory
WORKDIR /home/tester/bitcoin-counterparty-testnet-box


# expose rpc ports for the nodes to allow outside container access
EXPOSE 19001 19011 14000
CMD ["/bin/bash"]
