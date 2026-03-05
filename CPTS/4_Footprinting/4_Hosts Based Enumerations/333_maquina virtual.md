config setup
    charondebug="all"
    uniqueids=no

conn autopista
    keyexchange=ikev1
    aggressive=yes
    authby=psk
    ike=3des-sha1-modp1024
    left=%defaultroute
    leftid=ike@expressway.htb
    leftauth=psk
    right=10.129.10.60
    rightauth=psk
    xauth=client
    auto=add
