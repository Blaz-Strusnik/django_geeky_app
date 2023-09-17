# Use a base image with a known good mirror
FROM python:3.11.5-slim-bullseye

# Set environment variables
ARG BUILD_ENVIRONMENT=local
ENV PYTHONUNBUFFERED 1
ENV PYTHONDONTWRITEBYTECODE 1
ENV BUILD_ENV ${BUILD_ENVIRONMENT}

# Install system dependencies
RUN apt-get update && apt-get install --no-install-recommends -y \
    sudo git bash-completion nano ssh \
    build-essential \
    libpq-dev \
    gettext \
    && apt-get clean

# Create a dev-user and add to sudoers
RUN groupadd --gid 1000 dev-user \
    && useradd --uid 1000 --gid dev-user --shell /bin/bash --create-home dev-user \
    && echo "dev-user ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/dev-user \
    && chmod 0440 /etc/sudoers.d/dev-user

# Set the working directory
ARG APP_HOME=/app
WORKDIR ${APP_HOME}

# Copy application code to the container
COPY . ${APP_HOME}

# Entrypoint
COPY ./compose/production/django/entrypoint /entrypoint
RUN sed -i 's/\r$//g' /entrypoint
RUN chmod +x /entrypoint

# Start script for Django
COPY ./compose/local/django/start /start
RUN sed -i 's/\r$//g' /start
RUN chmod +x /start

# Start script for Celery worker
COPY ./compose/local/django/celery/worker/start /start-celeryworker
RUN sed -i 's/\r$//g' /start-celeryworker
RUN chmod +x /start-celeryworker

# Start script for Celery beat
COPY ./compose/local/django/celery/beat/start /start-celerybeat
RUN sed -i 's/\r$//g' /start-celerybeat
RUN chmod +x /start-celerybeat

# Start script for Celery Flower
COPY ./compose/local/django/celery/flower/start /start-flower
RUN sed -i 's/\r$//g' /start-flower
RUN chmod +x /start-flower

# Change Debian repository mirror
RUN sed -i 's/http:\/\/deb.debian.org/http:\/\/httpredir.debian.org/g' /etc/apt/sources.list

# Install Python dependencies
COPY ./requirements ${APP_HOME}/requirements
RUN pip install --no-cache-dir -r ./requirements/${BUILD_ENVIRONMENT}.txt

# Clean up
RUN apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /root/.cache

# Set permissions
RUN chown -R dev-user:dev-user ${APP_HOME}

# Switch to dev-user
USER dev-user

# Set entry point
ENTRYPOINT ["/entrypoint"]
