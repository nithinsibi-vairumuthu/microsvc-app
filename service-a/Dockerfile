FROM python:3.11-slim-bookworm

WORKDIR /app

# OS updates
RUN apt-get update && apt-get upgrade -y && rm -rf /var/lib/apt/lists/*

# Upgrade packaging tools FIRST
RUN pip install --no-cache-dir --upgrade pip setuptools wheel

# Copy and install deps
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy app
COPY . .

ENV APP_VERSION=dev
EXPOSE 5000
CMD ["python", "app.py"]
