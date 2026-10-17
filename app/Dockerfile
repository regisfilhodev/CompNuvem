FROM python:3.12-slim

WORKDIR /app

# 1) dependencias primeiro (cache de camadas)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 2) codigo depois
COPY app.py .
COPY templates ./templates

# 3) diretorio de uploads + usuario sem privilegios
RUN mkdir -p /app/uploads \
    && useradd --no-create-home --shell /usr/sbin/nologin appuser \
    && chown appuser:appuser /app/uploads
USER appuser

EXPOSE 5000
CMD ["gunicorn", "-b", "0.0.0.0:5000", "app:app"]
