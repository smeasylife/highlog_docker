# Nginx reverse proxy with SSL support
FROM nginx:alpine

# Remove default nginx config
RUN rm /etc/nginx/conf.d/default.conf

# Copy custom config
COPY nginx.conf /etc/nginx/nginx.conf

# Create directory for SSL certificates
RUN mkdir -p /etc/nginx/certs

# Copy SSL certificates (if available)
# Note: These files should be placed in ./certificates/ directory before building
# ./certificates/fullchain.pem -> /etc/nginx/certs/fullchain.pem
# ./certificates/privkey.pem -> /etc/nginx/certs/privkey.pem

# For development/testing without real certificates, we'll copy them at runtime
# In production, mount real certificates as volumes

# Expose ports
EXPOSE 80 443

# Start nginx
CMD ["nginx", "-g", "daemon off;"]
