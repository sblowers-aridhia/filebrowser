FROM alpine:latest

RUN apk --no-cache add curl

RUN apk update && apk upgrade && apk add bash

RUN apk --update add ca-certificates \
                     mailcap

RUN mkdir /lib64 && ln -s /lib/libc.musl-x86_64.so.1 /lib64/ld-linux-x86-64.so.2

# RUN curl -fsSL https://raw.githubusercontent.com/filebrowser/get/master/get.sh | bash

COPY docker/install_test /usr/bin/install_test

# COPY filebrowser/filebrowser /filebrowser
COPY filebrowser_build/filebrowser_build /filebrowser


# Required for the Workspace integrations
EXPOSE 8080

# Set up entrypoint
COPY docker/mini-app /usr/bin/mini-app
RUN chmod 755 /usr/bin/mini-app

# Run
ENTRYPOINT /usr/bin/mini-app
