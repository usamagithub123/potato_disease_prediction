# Base img
FROM python:3.11

# Set working directory
WORKDIR /app


# Copy dependencies
COPY requirements.txt .

# Install dependencies
RUN pip install -r requirements.txt

# Copy app code and model
COPY . .
COPY ./models ./models

# Expose port
EXPOSE 8000

# Command to run
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
