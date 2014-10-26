FROM ubuntu:trusty

MAINTAINER foostan ks@fstn.jp

EXPOSE 80

CMD while true; do ( echo "HTTP/1.0 200 Ok"; echo; echo `hostname` ) | nc -l 80; done
