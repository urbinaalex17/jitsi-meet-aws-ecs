FROM jitsi/web:stable-5142

# Copy custom configuration for Lefebvre, which have been extracted from jitsi/web:stable-5142 image
COPY interface_config.js /defaults
COPY all.css /usr/share/jitsi-meet/css
# Copy custom title
COPY title.html /usr/share/jitsi-meet/title.html
# favicon downloaded from https://assets.lefebvre.es/media/favicon.ico
COPY favicon.ico /usr/share/jitsi-meet/images
