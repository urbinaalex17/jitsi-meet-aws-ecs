FROM jitsi/web:stable-5142

# Copy custom configuration for Lefebvre, which have been extracted from jitsi/web:stable-5142 image
COPY interface_config.js /defaults
COPY all.css /usr/share/jitsi-meet/css
# Copy custom title
COPY title.html /usr/share/jitsi-meet/title.html
# favicon downloaded from https://assets.lefebvre.es/media/favicon.ico
COPY favicon.ico /usr/share/jitsi-meet/images
# Copy config.js and resources for redirect URL on hangup
COPY config.js /defaults
COPY static/thank-you-page.html /usr/share/jitsi-meet/static/close2.html
RUN mkdir /usr/share/jitsi-meet/static/css /usr/share/jitsi-meet/static/images
COPY static/css/* /usr/share/jitsi-meet/static/css
COPY static/images/* /usr/share/jitsi-meet/static/images