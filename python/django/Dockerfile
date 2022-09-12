FROM python:3-slim as base
FROM base as builder

WORKDIR /app

ENV PYTHONDONTWRITEBYTECODE 1 \
    PYTHONUNBUFFERED 1 

COPY pyproject.toml .
COPY poetry.lock .
RUN python3 -m pip install poetry &&\
    poetry export -f requirements.txt --without-hashes --output requirements.txt  &&\
    pip wheel --no-cache-dir --no-deps --wheel-dir /app/wheels -r requirements.txt


# final stage
FROM base as runner

WORKDIR /app
ENV PORT 8000
COPY --from=builder /app/wheels ./wheels
COPY --from=builder /app/requirements.txt .
COPY --from=builder /app/pyproject.toml .
COPY --from=builder /app/poetry.lock .
RUN apt-get -q update && apt-get -qy install netcat &&\
    pip install --no-cache ./wheels/*
COPY . .
CMD ["gunicorn", "-w", "4", "-b", ":\${PORT:-8000}", "-k", "uvicorn.workers.UvicornWorker", "config.asgi:application"]
