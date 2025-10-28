FROM python:3.10-slim

WORKDIR /app

COPY ./requirements.txt /app/requirements.txt

# Install dependencies required for compiling asyncpg, httptools, and psycopg2
RUN apt-get update && \
    apt-get install -y gcc g++ libpq-dev python3-dev build-essential && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Upgrade pip and install Python packages
RUN pip install --upgrade pip setuptools wheel && \
    pip install -r /app/requirements.txt && \
    rm -rf /root/.cache/pip

COPY . /app/

EXPOSE 8000


CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
