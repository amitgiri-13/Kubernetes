FROM busybox:latest

WORKDIR /app

# Copy code
COPY index.html .

# Expose port
EXPOSE 8000

# Run busybox httpd server
ENTRYPOINT ["httpd", "-f", "-v", "-p", "8000"]