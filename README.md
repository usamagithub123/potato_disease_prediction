================================================
FILE: potato_disease/Potato-Disease-Classification/README.md
================================================


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



================================================
FILE: potato_disease/Potato-Disease-Classification/docker-compose.yml
================================================
version: '3.9'
services:
  backend:
    build: ./api
    ports:
      - "8000:8000"
  frontend:
    build: ./frontend
    ports:
      - "3000:80"



================================================
FILE: potato_disease/Potato-Disease-Classification/api/Dockerfile
================================================
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



================================================
FILE: potato_disease/Potato-Disease-Classification/api/main.py
================================================
from fastapi import FastAPI, File, UploadFile
from fastapi.middleware.cors import CORSMiddleware
import uvicorn
import numpy as np
from io import BytesIO
from PIL import Image
import tensorflow as tf
import keras
import os


app = FastAPI()
import os
os.environ["TF_CPP_MIN_LOG_LEVEL"] = "2"
os.environ["TF_ENABLE_ONEDNN_OPTS"] = "0"


#to connect frontend and backend
origins = [
    "http://localhost",
    "http://localhost:3000",
]
app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Load Keras model directly
MODEL = keras.models.load_model("./models/1.keras")

# We can create diff. version of model and use it for different purposes 
# MODEL2 = keras.models.load_model("../saved_models/2.keras") 
# MODEL3 = keras.models.load_model("../saved_models/3.keras")
# MODEL$ = keras.models.load_model("./models/potatoes.h5")
CLASS_NAMES = ["Early Blight", "Late Blight", "Healthy"]

@app.get("/ping")
async def ping():
    return {"message": "Hello, I am alive!"} 

def read_file_as_image(data) -> np.ndarray:
    image = np.array(Image.open(BytesIO(data)))
    return image

@app.post("/predict")
async def predict(
    file: UploadFile = File(...)
):
    image = read_file_as_image(await file.read())
    img_batch = np.expand_dims(image, 0)
    
    predictions = MODEL.predict(img_batch)

    predicted_class = CLASS_NAMES[np.argmax(predictions[0])]
    confidence = np.max(predictions[0])
    return {
        'class': predicted_class,
        'confidence': float(confidence)
    }

if __name__ == "__main__":
    uvicorn.run(app, host='localhost', port=8001) 


================================================
FILE: potato_disease/Potato-Disease-Classification/api/requirements.txt
================================================
fastapi==0.116.1
uvicorn==0.23.2
numpy>=1.26,<1.27
pandas>=2.0,<2.1
tensorflow==2.20.0
keras==3.11.2
scikit-learn>=1.2,<1.3
opencv-python>=4.8,<4.9
Pillow>=10.0,<10.1
matplotlib>=3.7,<3.8



================================================
FILE: potato_disease/Potato-Disease-Classification/frontend/Dockerfile
================================================
# Build stage
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
# Use legacy OpenSSL if needed
ENV NODE_OPTIONS=--openssl-legacy-provider
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]




================================================
FILE: potato_disease/Potato-Disease-Classification/frontend/package.json
================================================
{
  "name": "photo",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "@material-ui/core": "^4.11.4",
    "@material-ui/icons": "^4.11.2",
    "axios": "^0.21.1",
    "material-ui-dropzone": "^3.5.0",
    "react": "^17.0.2",
    "react-dom": "^17.0.2",
    "react-scripts": "^4.0.3",
    "web-vitals": "^1.1.2"
  },
  "scripts": {
    "start": "react-scripts start --openssl-legacy-provider start",
    "build": "react-scripts build --openssl-legacy-provider build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
  "eslintConfig": {
    "extends": [
      "react-app",
      "react-app/jest"
    ]
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  }
}



================================================
FILE: potato_disease/Potato-Disease-Classification/frontend/public/index.html
================================================
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <link rel="icon" href="%PUBLIC_URL%/logo.PNG" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="theme-color" content="#000000" />
    <meta
      name="description"
      content="Web site created using create-react-app"
    />
    <link rel="apple-touch-icon" href="%PUBLIC_URL%/logo192.png" />
    <!--
      manifest.json provides metadata used when your web app is installed on a
      user's mobile device or desktop. See https://developers.google.com/web/fundamentals/web-app-manifest/
    -->
    <link rel="manifest" href="%PUBLIC_URL%/manifest.json" />
    <!--
      Notice the use of %PUBLIC_URL% in the tags above.
      It will be replaced with the URL of the `public` folder during the build.
      Only files inside the `public` folder can be referenced from the HTML.

      Unlike "/favicon.ico" or "favicon.ico", "%PUBLIC_URL%/favicon.ico" will
      work correctly both with client-side routing and a non-root public URL.
      Learn how to configure a non-root public URL by running `npm run build`.
    -->
    <title>Potato-Disease-Classification</title>
  </head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>

  </body>
</html>



================================================
FILE: potato_disease/Potato-Disease-Classification/frontend/public/robots.txt
================================================
# https://www.robotstxt.org/robotstxt.html
User-agent: *
Disallow:



================================================
FILE: potato_disease/Potato-Disease-Classification/frontend/src/App.js
================================================
import { ImageUpload } from "./home";

function App() {
  return <ImageUpload />;
}

export default App;



================================================
FILE: potato_disease/Potato-Disease-Classification/frontend/src/App.test.js
================================================
import { render, screen } from '@testing-library/react';
import App from './App';

test('renders learn react link', () => {
  render(<App />);
  const linkElement = screen.getByText(/learn react/i);
  expect(linkElement).toBeInTheDocument();
});



================================================
FILE: potato_disease/Potato-Disease-Classification/frontend/src/home.js
================================================
import { useState, useEffect } from "react";
import { makeStyles, withStyles } from "@material-ui/core/styles";
import AppBar from "@material-ui/core/AppBar";
import Toolbar from "@material-ui/core/Toolbar";
import Typography from "@material-ui/core/Typography";
import Avatar from "@material-ui/core/Avatar";
import Container from "@material-ui/core/Container";
import React from "react";
import Card from "@material-ui/core/Card";
import CardContent from "@material-ui/core/CardContent";
import { Paper, CardActionArea, CardMedia, Grid, TableContainer, Table, TableBody, TableHead, TableRow, TableCell, Button, CircularProgress } from "@material-ui/core";
import mylogo from "./mylogo.png";
import image from "./bg.png";
import { DropzoneArea } from 'material-ui-dropzone';
import { common } from '@material-ui/core/colors';
import Clear from '@material-ui/icons/Clear';




const ColorButton = withStyles((theme) => ({
  root: {
    color: theme.palette.getContrastText(common.white),
    backgroundColor: common.white,
    '&:hover': {
      backgroundColor: '#ffffff7a',
    },
  },
}))(Button);
const axios = require("axios").default;

const useStyles = makeStyles((theme) => ({
  grow: {
    flexGrow: 1,
  },
  clearButton: {
    width: "-webkit-fill-available",
    borderRadius: "15px",
    padding: "15px 22px",
    color: "#000000a6",
    fontSize: "20px",
    fontWeight: 900,
  },
  root: {
    maxWidth: 345,
    flexGrow: 1,
  },
  media: {
    height: 400,
  },
  paper: {
    padding: theme.spacing(2),
    margin: 'auto',
    maxWidth: 500,
  },
  gridContainer: {
    justifyContent: "center",
    padding: "4em 1em 0 1em",
  },
  mainContainer: {
    backgroundImage: `url(${image})`,
    backgroundRepeat: 'no-repeat',
    backgroundPosition: 'center',
    backgroundSize: 'cover',
    height: "93vh",
    marginTop: "8px",
  },
  imageCard: {
    margin: "auto",
    maxWidth: 400,
    height: 500,
    backgroundColor: 'transparent',
    boxShadow: '0px 9px 70px 0px rgb(0 0 0 / 30%) !important',
    borderRadius: '15px',
  },
  imageCardEmpty: {
    height: 'auto',
  },
  noImage: {
    margin: "auto",
    width: 400,
    height: "400 !important",
  },
  input: {
    display: 'none',
  },
  uploadIcon: {
    background: 'white',
  },
  tableContainer: {
    backgroundColor: 'transparent !important',
    boxShadow: 'none !important',
  },
  table: {
    backgroundColor: 'transparent !important',
  },
  tableHead: {
    backgroundColor: 'transparent !important',
  },
  tableRow: {
    backgroundColor: 'transparent !important',
  },
  tableCell: {
    fontSize: '22px',
    backgroundColor: 'transparent !important',
    borderColor: 'transparent !important',
    color: '#000000a6 !important',
    fontWeight: 'bolder',
    padding: '1px 24px 1px 16px',
  },
  tableCell1: {
    fontSize: '14px',
    backgroundColor: 'transparent !important',
    borderColor: 'transparent !important',
    color: '#000000a6 !important',
    fontWeight: 'bolder',
    padding: '1px 24px 1px 16px',
  },
  tableBody: {
    backgroundColor: 'transparent !important',
  },
  text: {
    color: 'white !important',
    textAlign: 'center',
  },
  buttonGrid: {
    maxWidth: "416px",
    width: "100%",
  },
  detail: {
    backgroundColor: 'white',
    display: 'flex',
    justifyContent: 'center',
    flexDirection: 'column',
    alignItems: 'center',
  },
  appbar: {
    background: '#be6a77',
    boxShadow: 'none',
    color: 'white'
  },
  loader: {
    color: '#be6a77 !important',
  }
}));
export const ImageUpload = () => {
  const classes = useStyles();
  const [selectedFile, setSelectedFile] = useState();
  const [preview, setPreview] = useState();
  const [data, setData] = useState();
  const [image, setImage] = useState(false);
  const [isLoading, setIsloading] = useState(false);
  let confidence = 0;

  const sendFile = async () => {
    if (image) {
      let formData = new FormData();
      formData.append("file", selectedFile);
      let res = await axios({
        method: "post",
        url: process.env.REACT_APP_API_URL,
        data: formData,
      });
      if (res.status === 200) {
        setData(res.data);
      }
      setIsloading(false);
    }
  }

  const clearData = () => {
    setData(null);
    setImage(false);
    setSelectedFile(null);
    setPreview(null);
  };

  useEffect(() => {
    if (!selectedFile) {
      setPreview(undefined);
      return;
    }
    const objectUrl = URL.createObjectURL(selectedFile);
    setPreview(objectUrl);
  }, [selectedFile]);

  useEffect(() => {
    if (!preview) {
      return;
    }
    setIsloading(true);
    sendFile();
  }, [preview]);

  const onSelectFile = (files) => {
    if (!files || files.length === 0) {
      setSelectedFile(undefined);
      setImage(false);
      setData(undefined);
      return;
    }
    setSelectedFile(files[0]);
    setData(undefined);
    setImage(true);
  };

  if (data) {
    confidence = (parseFloat(data.confidence) * 100).toFixed(2);
  }

  return (
    <React.Fragment>
      <AppBar position="static" className={classes.appbar}>
        <Toolbar>
          <Typography className={classes.title} variant="h6" noWrap>
             Potato Disease Classification
          </Typography>
          <div className={classes.grow} />
          <Avatar 
            src={mylogo}
            variant="square"
          ></Avatar>
        </Toolbar>
      </AppBar>
      <Container maxWidth={false} className={classes.mainContainer} disableGutters={true}>
        <Grid
          className={classes.gridContainer}
          container
          direction="row"
          justifyContent="center"
          alignItems="center"
          spacing={2}
        >
          <Grid item xs={12}>
            <Card className={`${classes.imageCard} ${!image ? classes.imageCardEmpty : ''}`}>
              {image && <CardActionArea>
                <CardMedia
                  className={classes.media}
                  image={preview}
                  component="image"
                  title="Contemplative Reptile"
                />
              </CardActionArea>
              }
              {!image && <CardContent className={classes.content}>
                <DropzoneArea
                  acceptedFiles={['image/*']}
                  dropzoneText={"Drag and drop an image of a potato plant leaf to process"}
                  onChange={onSelectFile}
                />
              </CardContent>}
              {data && <CardContent className={classes.detail}>
                <TableContainer component={Paper} className={classes.tableContainer}>
                  <Table className={classes.table} size="small" aria-label="simple table">
                    <TableHead className={classes.tableHead}>
                      <TableRow className={classes.tableRow}>
                        <TableCell className={classes.tableCell1}>Label:</TableCell>
                        <TableCell align="right" className={classes.tableCell1}>Confidence:</TableCell>
                      </TableRow>
                    </TableHead>
                    <TableBody className={classes.tableBody}>
                      <TableRow className={classes.tableRow}>
                        <TableCell component="th" scope="row" className={classes.tableCell}>
                          {data.class}
                        </TableCell>
                        <TableCell align="right" className={classes.tableCell}>{confidence}%</TableCell>
                      </TableRow>
                    </TableBody>
                  </Table>
                </TableContainer>
              </CardContent>}
              {isLoading && <CardContent className={classes.detail}>
                <CircularProgress color="secondary" className={classes.loader} />
                <Typography className={classes.title} variant="h6" noWrap>
                  Processing
                </Typography>
              </CardContent>}
            </Card>
          </Grid>
          {data &&
            <Grid item className={classes.buttonGrid} >

              <ColorButton variant="contained" className={classes.clearButton} color="primary" component="span" size="large" onClick={clearData} startIcon={<Clear fontSize="large" />}>
                Clear
              </ColorButton>
            </Grid>}
        </Grid >
      </Container >
    </React.Fragment >
  );
};



================================================
FILE: potato_disease/Potato-Disease-Classification/frontend/src/index.css
================================================
body {
	margin: 0;
	font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen',
		'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue',
		sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
}

code {
	font-family: source-code-pro, Menlo, Monaco, Consolas, 'Courier New',
		monospace;
}

.MuiDropzoneArea-root {
	background-color: transparent !important;
}
.MuiDropzoneArea-text, .MuiDropzoneArea-icon {
  color: white !important;
}


================================================
FILE: potato_disease/Potato-Disease-Classification/frontend/src/index.js
================================================
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import reportWebVitals from './reportWebVitals';

ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
);

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();



================================================
FILE: potato_disease/Potato-Disease-Classification/frontend/src/reportWebVitals.js
================================================
const reportWebVitals = onPerfEntry => {
  if (onPerfEntry && onPerfEntry instanceof Function) {
    import('web-vitals').then(({ getCLS, getFID, getFCP, getLCP, getTTFB }) => {
      getCLS(onPerfEntry);
      getFID(onPerfEntry);
      getFCP(onPerfEntry);
      getLCP(onPerfEntry);
      getTTFB(onPerfEntry);
    });
  }
};

export default reportWebVitals;


