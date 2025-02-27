# Use Python 3 as the base image
FROM python:3.10
# Set the working directory
WORKDIR /app
# Copy requirements.txt
COPY requirements.txt .
RUN pip install --upgrade pip
RUN pip install -r requirements.txt
# Copy the rest of the application code
COPY . .
# Run the application
CMD ["sh", "-c", "python3 main.py"]