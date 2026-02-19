FROM python:3.11-slim-bookworm

WORKDIR /app

# Install OS security updates
RUN apt-get update && apt-get upgrade -y && rm -rf /var/lib/apt/lists/*

# Copy and install dependencies first (better layer caching)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy app source
COPY . .

ENV APP_VERSION=dev

EXPOSE 5000

CMD ["python", "app.py"]
