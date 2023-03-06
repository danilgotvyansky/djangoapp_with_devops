FROM python:3
ENV PYTHONUNBUFFERED 1
WORKDIR /app
COPY requirements.txt ./
RUN pip install -r requirements.txt
COPY . ./
COPY wait-for-it.sh /code/
RUN python manage.py migrate
RUN python manage.py collectstatic --noinput
RUN DJANGO_SUPERUSER_PASSWORD=${DJANGO_SUPERUSER_PASSWORD} DJANGO_SUPERUSER_USERNAME=${DJANGO_SUPERUSER_USERNAME} DJANGO_SUPERUSER_EMAIL=${DJANGO_SUPERUSER_EMAIL} python manage.py createsuperuser --noinput
EXPOSE 8000
CMD ["/code/wait-for-it.sh", "${DB_HOST}:${DB_PORT}", "-t", "45", "python", "manage.py", "runserver", "0.0.0.0:8000"]
