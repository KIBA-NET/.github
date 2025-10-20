# KIBA

A device to sniff.

* [kiba-board](https://github.com/KIBA-NET/kiba-board): PCB design for a Raspberry PI with a SHT40 humidity/temperature sensor and up to 4 channels with a SGP40 VOC sensor and a CPS135B pressure sensor each, multiplexed with a TCA9546A
* [luftmeter](https://github.com/KIBA-NET/luftmeter): a plugin to publish the sensors data in the MADS framework
* [cps135b-i2c-driver](https://github.com/KIBA-NET/cps135b-i2c-driver): driver to interface with a CPS135B pressure sensor
* [kiba-server](https://github.com/KIBA-NET/kiba-server) first attempt for a standalone webapp to collect and show the measruements

## Getting started

Current distributed architecture runs the MADS broker and the data acquisition plugin in a RaspberryPi device (TartuPi), the Rerun viewer runs on a different device with desktop environment.

### TartuPi
* [MADS](https://github.com/pbosetti/MADS/releases/tag/v1.4.0)
```
# Install
curl -L -o mads.deb https://github.com/pbosetti/MADS/releases/download/v1.4.0/mads-v1.4.0-5-g76534a0-Linux-x86_64.deb

sudo apt install ./mads.deb

# Run
mads broker
```
* [LUFTMETER-PLUGIN](https://github.com/KIBA-NET/luftmeter)
```
# Install
git clone https://github.com/KIBA-NET/luftmeter
cmake -Bbuild -DCMAKE_INSTALL_PREFIX="$(mads -p)"
cmake --build build -j4
sudo cmake --install build

# Run
mads source luftmeter.plugin
```

### Linux Desktop
* [MADS RERUNNER PLUGIN](https://github.com/MADS-NET/rerunner_plugin/tree/main) 
```
# Rerun dependencies
sudo apt-get -y install \
    libclang-dev \
    libatk-bridge2.0 \
    libfontconfig1-dev \
    libfreetype6-dev \
    libglib2.0-dev \
    libgtk-3-dev \
    libssl-dev \
    libxcb-render0-dev \
    libxcb-shape0-dev \
    libxcb-xfixes0-dev \
    libxkbcommon-dev \
    patchelf \

sudo add-apt-repository ppa:kisak/kisak-mesa
sudo apt-get update
sudo apt-get install -y mesa-vulkan-drivers
# If Ubuntu 24
sudo apt install libxkbcommon-x11-0

sudo pip3 install rerun-sdk=0.25.1 --break-system-packages

# Build and install plugin
git clone https://github.com/MADS-NET/rerunner_plugin/tree/main
cmake -Bbuild -DCMAKE_INSTALL_PREFIX="$(mads -p)"
cmake --build build -j4
sudo cmake --install build

# Run
mads sink rerunner.plugin -s tcp://<tartupi-ip>:9092
```

## MADS INI settings 
Add this to the Plugin section of `/usr/local/etc/mads.ini`
```
[luftmeter]
period = 500
pub_topic = "sensors"
activate_channels = "0101"

[rerunner]
sub_topic = ["sensors"]
keypaths = ["sensors/rh", "sensors/temp", "sensors/voc", "sensors/pres"]
```