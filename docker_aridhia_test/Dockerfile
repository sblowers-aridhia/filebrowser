FROM alpine:latest

RUN apk --no-cache add curl

RUN apk update && apk upgrade && apk add bash

RUN apk --update add ca-certificates \
                     mailcap

RUN curl -fsSL https://raw.githubusercontent.com/filebrowser/get/master/get.sh | bash

COPY docker/install_test /usr/bin/install_test

# COPY filebrowser/filebrowser /filebrowser
COPY filebrowser_build/filebrowser_build /usr/local/bin/filebrowser

# COPY docker/config.json /.filebrowser.json
# RUN touch /database.db

# RUN filebrowser config import /.filebrowser.json

# RUN filebrowser users add "admin" "admin"

# RUN filebrowser config init

# RUN filebrowser config set --port=8080 && \
#	filebrowser config set --baseurl="" && \
#	filebrowser config set --address="" && \
#	filebrowser config set --log="stdout" && \
#	filebrowser config set --root="/home/workspace/files" && \
#	filebrowser config set --auth.method=noauth
	
# RUN filebrowser users add "admin" "admin"

# Required for the Workspace integrations
EXPOSE 8080

# Set up entrypoint
COPY docker/mini-app /usr/bin/mini-app
RUN chmod 755 /usr/bin/mini-app

# Run
ENTRYPOINT /usr/bin/mini-app
