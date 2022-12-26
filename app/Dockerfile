FROM python:alpine

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED 1
ENV DJANGO_SUPERUSER_PASSWORD admin

RUN mkdir /app
WORKDIR /app

RUN apk add --update --no-cache postgresql-client
RUN apk add --update --no-cache --virtual .tmp-build-deps \
      gcc libc-dev linux-headers postgresql-dev musl-dev zlib zlib-dev

RUN pip install --upgrade pip
RUN pip install django gunicorn
COPY requirements.txt /app/

RUN pip install -r requirements.txt
RUN apk del .tmp-build-deps

COPY . /app/

CMD while ! python3 manage.py sqlflush > /dev/null 2>&1 ; do sleep 1 ; done && \
    python3 manage.py makemigrations --noinput && \
    python3 manage.py migrate --noinput && \
    python3 manage.py collectstatic --noinput && \
    python3 manage.py createsuperuser --user admin --email admin@localhost --noinput; \
    gunicorn -b 0.0.0.0:8000 config.wsgi
