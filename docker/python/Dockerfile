FROM python:3.8-alpine
COPY ./docker/python/app /app
WORKDIR /app
RUN pip install -r ./requirements.txt
CMD ["python","./fl.py"]
