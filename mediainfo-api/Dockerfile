FROM python:3.12-alpine
RUN apk add --no-cache mediainfo
WORKDIR /app
COPY app.py .
ENV ALLOWED_ROOT=/media \
    PORT=8765 \
    MEDIAINFO_TIMEOUT=60 \
    PYTHONUNBUFFERED=1
EXPOSE 8765
HEALTHCHECK --interval=60s --timeout=5s CMD wget -qO- http://127.0.0.1:${PORT}/health || exit 1
CMD ["python", "app.py"]
