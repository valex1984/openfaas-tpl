FROM openfaas/of-watchdog:0.9.11 as watchdog
FROM python:3.7-slim

COPY --from=watchdog /fwatchdog /usr/bin/fwatchdog
RUN chmod +x /usr/bin/fwatchdog

# Add non root user
RUN addgroup --system app && adduser app --system --ingroup app --home /home/app

USER app

ENV PATH=$PATH:/home/app/.local/bin

WORKDIR /home/app/

COPY index.py           .
COPY pip.conf           /etc
COPY requirements.txt   .
USER root
RUN pip install -r requirements.txt
USER app

RUN mkdir -p function
RUN touch ./function/__init__.py
WORKDIR /home/app/function/
COPY function/requirements.txt	.
RUN pip install --user -r requirements.txt

USER root
COPY function/   .
RUN chown -R app:app ../

ARG TEST_COMMAND=tox
ARG TEST_ENABLED=true
RUN /bin/bash -c '[ "$TEST_ENABLED" == "false" ] &&  echo "skipping tests" \
    || eval "$TEST_COMMAND"'

WORKDIR /home/app/

USER app

# Set up of-watchdog for HTTP mode
ENV fprocess="python index.py"
ENV cgi_headers="true"
ENV mode="http"
ENV upstream_url="http://127.0.0.1:5000"

HEALTHCHECK --interval=5s CMD [ -e /tmp/.lock ] || exit 1

CMD ["fwatchdog"]