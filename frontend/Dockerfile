FROM nginx:alpine

# Copy React production build
COPY build /usr/share/nginx/html

# Copy custom Nginx configuration
COPY nginx.conf /etc/nginx/nginx.conf

# Configure permissions for non-root Nginx
RUN chown -R nginx:nginx /usr/share/nginx/html \
    && chown -R nginx:nginx /var/cache/nginx \
    && chown -R nginx:nginx /var/log/nginx \
    && touch /var/run/nginx.pid \
    && chown nginx:nginx /var/run/nginx.pid

# Run Nginx as non-root user
USER nginx

EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]
