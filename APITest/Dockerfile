FROM python:3-alpine

# WebApp server address and port can be set at image build time and/or at run time.
ARG WEBAPP_ADDR
ENV WEBAPP_ADDR ${WEBAPP_ADDR:-http://localhost}
ARG WEBAPP_PORT
ENV WEBAPP_PORT ${WEBAPP_PORT:-80}

WORKDIR /usr/src/app

COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

COPY ./*.py ./

CMD python ./TestClickCountAPI.py --webappAddr ${WEBAPP_ADDR} --webappPort ${WEBAPP_PORT}
