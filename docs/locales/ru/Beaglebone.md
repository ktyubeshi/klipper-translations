# Плата BeagleBone

В этом документе описывается процесс запуска Klipper на Beaglebone PRU.

## Сборка образа ОС

Начните с установки [Debian 9.9 2019-08-03 4GB SD IoT](https://beagleboard.org/latest-images ) изображение. Изображение можно запустить либо с карты micro-SD, либо со встроенного eMMC. Если вы используете eMMC, установите его в eMMC сейчас, следуя инструкциям по приведенной выше ссылке.

Затем подключитесь по ssh к машине Beaglebone (`ssh debian@beaglebone` -- пароль `temppwd`) и установите Klipper, выполнив следующие команды:

```
git clone https://github.com/Klipper3d/klipper
./klipper/scripts/install-beaglebone.sh
```

## Установите Octoprint

One may then install Octoprint:

```
git clone https://github.com/foosel/OctoPrint.git
cd OctoPrint/
virtualenv venv
./venv/bin/python setup.py install
```

И настройте OctoPrint на запуск при загрузке:

```
sudo cp ~/OctoPrint/scripts/octoprint.init /etc/init.d/octoprint
sudo chmod +x /etc/init.d/octoprint
sudo cp ~/OctoPrint/scripts/octoprint.default /etc/default/octoprint
sudo update-rc.d octoprint defaults
```

It is necessary to modify OctoPrint's **/etc/default/octoprint** configuration file. One must change the `OCTOPRINT_USER` user to `debian`, change `NICELEVEL` to `0`, uncomment the `BASEDIR`, `CONFIGFILE`, and `DAEMON` settings and change the references from `/home/pi/` to `/home/debian/`:

```
sudo nano /etc/default/octoprint
```

Then start the Octoprint service:

```
sudo systemctl start octoprint
```

Make sure the OctoPrint web server is accessible - it should be at: <http://beaglebone:5000/>

## Building the micro-controller code

To compile the Klipper micro-controller code, start by configuring it for the "Beaglebone PRU":

```
cd ~/klipper/
make menuconfig
```

To build and install the new micro-controller code, run:

```
sudo service klipper stop
make flash
sudo service klipper start
```

It is also necessary to compile and install the micro-controller code for a Linux host process. Configure it a second time for a "Linux process":

```
make menuconfig
```

Then install this micro-controller code as well:

```
sudo service klipper stop
make flash
sudo service klipper start
```

## Remaining configuration

Complete the installation by configuring Klipper and Octoprint following the instructions in the main [Installation](Installation.md#configuring-klipper) document.

## Printing on the Beaglebone

Unfortunately, the Beaglebone processor can sometimes struggle to run OctoPrint well. Print stalls have been known to occur on complex prints (the printer may move faster than OctoPrint can send movement commands). If this occurs, consider using the "virtual_sdcard" feature (see [Config Reference](Config_Reference.md#virtual_sdcard) for details) to print directly from Klipper.
