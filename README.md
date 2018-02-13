# steamlinkVPN
OpenVPN 2.4.4 setup for a [Steam Link](http://store.steampowered.com/app/353380/Steam_Link/)

Tested on ubuntu server 16.04 with generic dev tools (TODO: document setup from fresh install)

## General Setup
Within a work folder:

- Download the [Steam Link SDK](https://github.com/ValveSoftware/steamlink-sdk)
- `$ source steamlink-sdk/setenv.sh`
- `$ export PREFIX=<work folder>/deps`
- `$ export CROSS=armv7a-cros-linux-gnueabi`

### Set up LZO

- `$ wget http://www.oberhumer.com/opensource/lzo/download/lzo-2.09.tar.gz`
- `$ tar xzf lzo-2.09.tar.gz && cd lzo-2.09`
- `$ ./configure $STEAMLINK_CONFIGURE_OPTS --enable-static --prefix=$PREFIX`
- `$ make && make install`

At this point you should have some files in `<work folder>/deps`.

### Set up OpenSSL

- `$ wget https://www.openssl.org/source/openssl-1.0.2j.tar.gz`
- `$ tar xzf openssl-1.0.2j.tar.gz && cd openssl-1.0.2j`
- `$ ./Configure gcc no-ssl2 no-shared -static --prefix=$PREFIX --cross-compile-prefix=$CROSS-`
- `$ make && make install`

### Set up OpenVPN

- `$ wget https://swupdate.openvpn.org/community/releases/openvpn-2.4.4.tar.gz`
- `$ tar xzf openvpn-2.4.4.tar.gz && cd openvpn-2.4.4`
- `$ sed -i -e 's/\/dev\/net\/tun/\/dev\/tun/g' src/openvpn/tun.c` (this is bad, but I wasn't able to get the --dev-node to work properly when the utility is built normally.
- `$ ./configure --target=$CROSS --host=$CROSS --prefix=$PREFIX --disable-server --enable-static --disable-shared --disable-debug --disable-plugins OPENSSL_SSL_CFLAGS="-I$PREFIX/include" OPENSSL_SSL_LIBS="-L$PREFIX/lib -lssl" OPENSSL_CRYPTO_CFLAGS="-I$PREFIX/include" OPENSSL_CRYPTO_LIBS="-L$PREFIX/lib -lcrypto" LZO_CFLAGS="-I$PREFIX/include" LZO_LIBS="-L$PREFIX/lib -llzo2" IFCONFIG=/bin/ifconfig ROUTE=/bin/route`
- `$ make LIBS="-all-static" && make install`

At this point a working `openvpn` binary for your steam link should exist at `deps/sbin/openvpn`.

### Setup on the steam link

