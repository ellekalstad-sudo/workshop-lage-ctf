# Minimal Flask skeleton — replace with your own application.
# Build rules (see FRAMEWORK.md, Phase 2): one-command-up, self-contained, config via env.
FROM python:3.12-slim

WORKDIR /app

# Install dependencies first for better layer caching
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Application source (app/main.py + app/static/)
COPY app/ ./app/

EXPOSE 8101

CMD ["python", "app/main.py"]
