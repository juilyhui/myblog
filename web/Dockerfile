FROM scratch
MAINTAINER izgnod <izgnod@gmail.com>
ADD ./rootfs.tar /
ADD ./myblog/bin/ /myblog/bin/
WORKDIR /myblog/bin/
EXPOSE 9899
CMD ["./pugo", "server"]
