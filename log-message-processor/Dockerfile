FROM python:3.9
WORKDIR /app
COPY requirements.txt .
RUN grep -v "py_zipkin" requirements.txt | pip install -r /dev/stdin
RUN pip install py_zipkin==0.20.0
COPY . .
CMD ["sh", "-c", "python3 main.py"]