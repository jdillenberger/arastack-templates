FROM ghcr.io/openclaw/openclaw:latest

USER root

# Add python3 plus the dependencies of the caldav skill
# (https://clawhub.ai/skills/caldav-skill). PEP 668 (externally-managed)
# requires --break-system-packages on Debian 12; this image is dedicated to
# OpenClaw and not intended for ad-hoc Python work, so the override is fine.
RUN apt-get update \
 && apt-get install -y --no-install-recommends \
      python3 \
      python3-pip \
 && pip3 install --no-cache-dir --break-system-packages \
      caldav \
      icalendar \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

USER node
