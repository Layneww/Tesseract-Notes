# Build tesseract from source
### install required library
```
# install tesseract 4.x on Ubuntu 18.xx
apt install tesseract-ocr
# Developer Tools
apt install libtesseract-dev
```
if such error message appears,
```
dpkg: warning: 'ldconfig' not found in PATH or not executable.
dpkg: warning: 'start-stop-daemon' not found in PATH or not executable.
dpkg: error: 2 expected programs not found in PATH or not executable.
Note: root's PATH should usually contain /usr/local/sbin, /usr/sbin and /sbin.
E: Sub-process /usr/bin/dpkg returned an error code (2)
```
run
```export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin```
```
sudo apt-get install g++ # or clang++ (presumably)
sudo apt-get install autoconf automake libtool
sudo apt-get install autoconf-archive
sudo apt-get install pkg-config
sudo apt-get install libpng-dev
sudo apt-get install libjpeg8-dev
sudo apt-get install libtiff5-dev
sudo apt-get install zlib1g-dev

# for training tools
sudo apt-get install libicu-dev
sudo apt-get install libpango1.0-dev
sudo apt-get install libcairo2-dev
```


### build leptonica 1.76.0 from source
```
wget http://www.leptonica.org/source/leptonica-1.76.0.tar.gz
gunzip leptonica-1.76.0.tar.gz
tar -xvf leptonica-1.76.0.tar
cd leptonica-1.76.0
./configure
make
sudo make install
```

### build tesseract 4.0 alpha from source
```
# library needed
apt-get install autoconf-archive automake g++ libtool libleptonica-dev make pkg-config
apt-get install asciidoc
apt-get install libpango1.0-dev

# clone the 4.00.00alpha branch
git clone https://github.com/tesseract-ocr/tesseract.git --branch 4.00.00alpha --single-branch tesseract-ocr
# clone the master branch
git clone https://github.com/tesseract-ocr/tesseract.git tesseract-ocr

cd tesseract
./autogen.sh
autoreconf -i
./configure --enable-debug
LDFLAGS="-L/usr/local/lib" CFLAGS="-I/usr/local/include" make
sudo make install
sudo ldconfig

# build training tool
make training
sudo make training-install
# optional:
make ScrollView.jar
export SCROLLVIEW_PATH=$PWD/java
```
