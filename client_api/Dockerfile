# Use official Python base image
FROM python:3.10-slim

# Set working directory inside the container
WORKDIR /app

# Install MongoDB client tools and other dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    wget gnupg curl \
    gcc python3-dev \
    && wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | apt-key add - \
    && echo "deb http://repo.mongodb.org/apt/debian buster/mongodb-org/6.0 main" | tee /etc/apt/sources.list.d/mongodb-org-6.0.list \
    && apt-get update \
    && apt-get install -y mongodb-mongosh \
    && apt-get remove -y wget gnupg \
    && rm -rf /var/lib/apt/lists/*

# Copy dependency file first
COPY requirements.txt .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy the FastAPI application files
COPY . /app/

# Check if the files are copied
RUN ls -la /app

# Expose the application port
EXPOSE 80

# Run the FastAPI app with Uvicorn
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "80"]
