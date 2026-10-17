# Proxy image inherited from the previous team.
# Lightweight Nginx proxy image
FROM nginx:alpine
# Copy the custom Nginx configuration
COPY nginx.conf /etc/nginx/nginx.conf
# Allow the non-root nginx user to write runtime files.
RUN mkdir -p /var/cache/nginx /var/run /tmp/nginx && \
    chown -R nginx:nginx /var/cache/nginx /var/run /tmp/nginx

# Run the container as a non-root user.
USER nginx
EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]
