FROM python:3.8-slim-buster
RUN apt-get update && apt-get install -y docker.io
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 5000
CMD ["python3.8", "infra_app.py"]
