FROM python:3
ENV PYTHONUNBUFFERED 1
WORKDIR /app
COPY requirements.txt ./
COPY . ./
RUN pip install -r requirements.txt
COPY wait-for-it.sh /code/
EXPOSE 8000
