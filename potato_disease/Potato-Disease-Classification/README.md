

# 🥔 Potato Disease Classification

A full-stack deep learning application that classifies potato plant diseases from leaf images using a trained neural network model. Built with **FastAPI backend** and **React frontend**, containerized with **Docker**.

---

## 🌟 Features

* **Image Classification**: Upload potato leaf images to detect Early Blight, Late Blight, or Healthy plants
* **Real-time Predictions**: Fast inference using a pre-trained Keras/TensorFlow model
* **Responsive UI**: Material-UI based React frontend with drag-and-drop upload
* **RESTful API**: FastAPI backend with CORS support
* **Dockerized**: Complete containerization for easy deployment
* **Confidence Scores**: Returns prediction confidence percentages

---

## 🏗️ Architecture

```
Potato-disease/
├── api/                 # FastAPI Backend
│   ├── models/          # Keras model files
│   ├── main.py          # FastAPI application
│   ├── requirements.txt # Python dependencies
│   └── Dockerfile       # Backend container setup
├── frontend/            # React Frontend
│   ├── src/
│   ├── public/
│   ├── package.json
│   └── Dockerfile       # Frontend container setup
└── docker-compose.yml   # Multi-container orchestration
```

---

## 🚀 Quick Start

### Prerequisites

* Docker and Docker Compose
* Git

### Installation

```bash
git clone https://github.com/Ahamed-Safnas/Potato-Disease-Classification.git
cd Potato-Disease-Classification
```

---

## 🐳 Option 1: Run with Docker Compose (Recommended)

1. **Start with Docker Compose**

   ```bash
   docker-compose up -d --build
   ```

2. **Access the application**

   * Frontend → [http://localhost:3000](http://localhost:3000)
   * Backend API → [http://localhost:8000](http://localhost:8000)
   * API Documentation → [http://localhost:8000/docs](http://localhost:8000/docs)

---

## 🐳 Option 2: Build & Run Services Separately (Frontend + Backend)

### 🔹 Backend (FastAPI)

**Build Image**

```bash
docker build -t potato-backend ./api
```

**Run Container**

```bash
docker run -d -p 8000:8000 --name potato-backend potato-backend
```

* Backend API → [http://localhost:8000](http://localhost:8000)
* API Docs → [http://localhost:8000/docs](http://localhost:8000/docs)

---

### 🔹 Frontend (React)

**Build Image**

```bash
docker build -t potato-frontend ./frontend
```

**Run Container**

```bash
docker run -d -p 3000:80 --name potato-frontend potato-frontend
```

* Frontend UI → [http://localhost:3000](http://localhost:3000)

---

## 📖 API Endpoints

### `GET /ping`

Health check endpoint

```bash
curl http://localhost:8000/ping
```

### `POST /predict`

Upload potato leaf image for classification

```bash
curl -X POST -F "file=@path_to_image.jpg" http://localhost:8000/predict
```

**Response Example**

```json
{
  "class": "Early Blight",
  "confidence": 0.9567
}
```

---

## 🛠️ Manual Development Setup (Without Docker)

### Backend Setup

```bash
cd api
python -m venv venv
source venv/bin/activate  # Linux/Mac
venv\Scripts\activate     # Windows
pip install -r requirements.txt
uvicorn main:app --reload --port 8000
```

### Frontend Setup

```bash
cd frontend
npm install
npm start
```

---

## 📊 Model Information

* **Framework**: TensorFlow/Keras
* **Classes**: Early Blight, Late Blight, Healthy
* **Input**: Potato leaf images
* **Output**: Classification with confidence score

---

## 🎯 Usage

1. Open the web application at [http://localhost:3000](http://localhost:3000)
2. Drag and drop a potato leaf image or click to browse
3. View classification results with confidence percentage
4. Use the clear button to analyze another image

---

## 🐳 Useful Docker Commands

**Using Docker Compose**

```bash
docker-compose build backend
docker-compose up frontend
docker-compose logs -f backend
docker-compose down
```

**Using Separate Containers**

```bash
# Stop containers
docker stop potato-backend potato-frontend

# Remove containers
docker rm potato-backend potato-frontend

# Remove images
docker rmi potato-backend potato-frontend
```

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request


---

## 🙏 Acknowledgments

* Potato leaf dataset providers
* TensorFlow and FastAPI communities
* Material-UI React components

---

## 🆘 Support

For support, please open an issue in the GitHub repository or contact the maintainers.

---

⚡ This README now gives you **two clear ways** to run your app:

* **Option 1:** Use `docker-compose` for both containers at once
* **Option 2:** Build & run **frontend/backend separately**

---
