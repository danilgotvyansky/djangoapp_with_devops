FROM python:3
ENV PYTHONUNBUFFERED 1
WORKDIR /app
COPY requirements.txt ./
RUN pip install -r requirements.txt
COPY . ./
RUN python manage.py collectstatic --noinput
COPY wait-for-it.sh /code/backend/
EXPOSE 8000