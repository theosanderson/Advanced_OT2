## Installing Raspbian on the OT2

** THIS IS NOW OUT OF DATE **

`sudo apt-get install libatlas-base-dev` for numpy
`python3 -m venv ot_env`
`git clone https://github.com/Opentrons/opentrons.git`
`python setup.py  install`

```
158  apt download python3-libgpiod
  159  ls
  160  dpkg -x python3-libgpiod_1.2-3+rpi1_armhf.deb libgpiod
  161  cd libgpiod/
  162  ls
  163  cd usr/
  164  ls
  165  cd lib/
  166  ls
  167  cd python3/
  168  ls
  169  cd dist-packages/
  170  ls
  171  cp gpiod.cpython-37m-arm-linux-gnueabihf.so ~/ot_env/lib/python3.7/site-packages/

```
