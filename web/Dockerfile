FROM python:3.13-slim AS builder

RUN apt-get update && \
    apt-get upgrade -y

RUN useradd --create-home customuser
USER customuser
WORKDIR /home/customuser

ENV VIRTUALENV=/home/customuser/venv
RUN python3 -m venv $VIRTUALENV
ENV PATH="$VIRTUALENV/bin:$PATH"

COPY --chown=customuser pyproject.toml constraints.txt ./
RUN python -m pip install --upgrade pip setuptools && \
    python -m pip install --no-cache-dir -c constraints.txt ".[dev]"

COPY --chown=customuser src/ src
COPY --chown=customuser tests/ tests/

RUN python -m pip install . -c constraints.txt && \
    python -m pytest tests/unit/ && \
    python -m flake8 src/ && \
    python -m isort src/ --check && \
    python -m black src/ --check --quiet && \
    python -m pylint src/ --disable=C0114,C0116,R1705 && \
    python -m bandit -r src/ --quiet && \
    python -m pip wheel --wheel-dir dist/ . -c constraints.txt

FROM python:3.13-slim

RUN apt-get update && \
    apt-get upgrade -y

RUN useradd --create-home customuser
USER customuser
WORKDIR /home/customuser

ENV VIRTUALENV=/home/customuser/venv
RUN python3 -m venv $VIRTUALENV
ENV PATH="$VIRTUALENV/bin:$PATH"

COPY --from=builder /home/customuser/dist/page_tracker*.whl /home/customuser

RUN python -m pip install --upgrade pip setuptools && \
    python -m pip install --no-cache-dir page_tracker*.whl

CMD [ "flask", "--app", "page_tracker.app", "run", \
     "--host", "0.0.0.0", "--port", "5000"]
