## docker run example:

docker run -d --name coredns \
           -m 256m \
           --log-opt max-size=16m \
           --restart=always \
           -v /etc/localtime:/etc/localtime \
           -v PATH_TO_Corefile:/Corefile \
           -v PATH_TO_ADBLOCK_HOST:/etc/hosts \
           -p 53:53/tcp \
           -p 53:53/udp \
           -p 9053:9053/tcp \
           ruijzhan/coredns:armv7l

