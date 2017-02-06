# Installation of Optware IPKG
Synology DS216j IPKG installation

# Introduction
The CPU of DS 216 j is "Marvell Armada 385 88 F 6820", although there is no optware-bootstrap for Marvell Armada, it seems that Marvell Kirkwood's optware-bootstrap can be diverted as it is compatible.

## Note on compatibility
It appear that binaries aren't that much/all compatible. The reason must be that the cs08q1armel toolchain doesn't have updated materials. If you encounter issues you should consider setup a chroot environment instead.[*](https://github.com/trepmag/ds213j-optware-bootstrap)

1. Connect with the appropriate SSH client (PuTTY, Terminal)
2. Get root privileges before starting.
`sudo -i`
3. Create optware root directory (do these one line at a time).

    ```
mkdir /volume1/@optware
mkdir /opt
mount -o bind /volume1/@optware /opt
    ```
4. Set up IPKG.

    ```
feed=http://ipkg.nslu2-linux.org/feeds/optware/cs08q1armel/cross/unstable
ipk_name=`wget -qO- $feed/Packages | awk '/^Filename: ipkg-opt/ {print $2}'`
wget $feed/$ipk_name
tar -xOvzf $ipk_name ./data.tar.gz | tar -C / -xzvf -
mkdir -p /opt/etc/ipkg
echo "src cross $feed" > /opt/etc/ipkg/feeds.conf
    ```

5. Set PATH: Open: /etc/profile (with vi, nano etc.) and add the following to the last line.
`PATH=/opt/bin:/opt/sbin:$PATH` <-- make sure to add $PATH to the end.

6. Set the init script so that it can be used even after rebooting.

  1. Create /etc/rc.local with the following contents and attach execution privilege (chmod 755).

    ```
#!/bin/sh

# Optware setup
[ -x /etc/rc.optware ] && /etc/rc.optware start
    ```

  1. Create /etc/rc.optware with the following contents and attach execute privilege (chmod 755).

    ```
#!/bin/sh

if test -z "${REAL_OPT_DIR}"; then
# next line to be replaced according to OPTWARE_TARGET
REAL_OPT_DIR=/volume1/@optware
fi

case "$1" in
    start)
        echo "Starting Optware."
        if test -n "${REAL_OPT_DIR}"; then
            if ! grep ' /opt ' /proc/mounts >/dev/null 2>&1 ; then
                mkdir -p /opt
                mount -o bind ${REAL_OPT_DIR} /opt
            fi  
        fi
    [ -x /opt/etc/rc.optware ] && /opt/etc/rc.optware
    ;;
    reconfig)
    true
    ;;
    stop)
        echo "Shutting down Optware."
    true
    ;;
    *)
        echo "Usage: $0 {start|stop|reconfig}"
        exit 1
esac

exit 0
    ```

1. IPKG can now be used.

    ```
ipkg -v
ipkg version 0.99.163
    ```

## Other
Update the ipkg list of available packages : `ipkg update`
Install a package using the command : `ipkg install <packagename>`
You can list all available packages with : `ipkg list`

### Sources
http://jasmin.sakura.ne.jp/blog/0244
http://www.ingmarverheij.com/how-to-install-ipkg-on-synology-nas-ds212/
https://github.com/trepmag/ds213j-optware-bootstrap
