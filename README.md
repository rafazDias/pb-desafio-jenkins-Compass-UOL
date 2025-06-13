
# Projeto Desafio Jenkins - Documentação


## Fase 1: Preparação do projeto

  

1.1 - Crie uma branch chamada `dev` onde iremos trabalhar no nosso projeto.
  
1.2 - Crie 3 pastas no seu repositório para organizar o projeto:

-  `backend`

-  `frontend`

-  `k8s` (para os arquivos do Kubernetes)

1.3 - Crie uma conta no Docker Hub para hospedar suas imagens.

  

1.4 - Inicie o Kubernetes localmente. Exemplo usando Minikube:

```bash
minikube  start
````

1.5 - Configure as credentials que utilizaremos no Jenkins:
* Crie as credentials para fazer login no Docker Hub.

* Crie as credentials para o arquivo de configuração do Kubernetes.  

Pegue o arquivo de configuração com:

```bash
kubectl config view --minify --flatten --raw > kubeconfig-jenkins.yaml
```
Insira esse arquivo nas credentials do Jenkins, nomeando como `kubeconfig`.

  ---

## Fase 2: Containerização com Docker  

### Backend

Na pasta `backend` crie um arquivo `main.py` com a seguinte aplicação Python usando FastAPI:

```python
from fastapi import FastAPI
from fastapi.responses import JSONResponse, RedirectResponse
from fastapi.middleware.cors import CORSMiddleware
from datetime import datetime
import random
import httpx

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Ou especifique ['http://localhost:3000'] se preferir
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 1️⃣ Endpoint que retorna uma cor aleatória para mudar a cor da página
@app.get("/color")
async def get_random_color():
    colors = ["#FF5733", "#33FF57", "#3357FF", "#F333FF", "#33FFF3"]
    return {"color": random.choice(colors)}

# 2️⃣ Endpoint que retorna uma imagem aleatória de gato
@app.get("/cat")
async def get_random_cat_image():
    async with httpx.AsyncClient() as client:
        response = await client.get("https://api.thecatapi.com/v1/images/search")
        if response.status_code == 200:
            data = response.json()
            image_url = data[0]['url']
            return {"cat_image_url": image_url}
        return JSONResponse(content={"error": "Failed to fetch cat image"}, status_code=500)

# 3️⃣ Endpoint que retorna uma imagem aleatória (via https://picsum.photos)
@app.get("/random-photo")
async def get_random_photo():
    width = random.randint(200, 600)
    height = random.randint(200, 600)
    photo_url = f"https://picsum.photos/{width}/{height}"
    return {"random_photo_url": photo_url}

# 4️⃣ Endpoint que retorna o horário atual
@app.get("/time")
async def get_current_time():
    now = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    return {"current_time": now}

@app.get("/scare")
async def scare():
    scare_images = [
        "https://media.giphy.com/media/l0MYt5jPR6QX5pnqM/giphy.gif",  # Exemplo de imagem de susto
        "https://media.giphy.com/media/26xBI73gWquCBBCDe/giphy.gif"
    ]
    random_scare = random.choice(scare_images)
    return {"scare_image_url": random_scare}


@app.get("/lookalike")
async def lookalike():
    lookalike_images = [
        "https://randomuser.me/api/portraits/men/1.jpg",
        "https://randomuser.me/api/portraits/women/1.jpg",
        "https://randomuser.me/api/portraits/lego/1.jpg"
    ]
    random_lookalike = random.choice(lookalike_images)
    return {"lookalike_image_url": random_lookalike}
```

  

### Requisitos do backend (`requirements.txt`):

```
fastapi
uvicorn
httpx
pydantic
```

### Dockerfile do backend:

```dockerfile

FROM python:3.9

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]

```
---

  

### Frontend

Na pasta `frontend`, crie o arquivo `index.html` com o seguinte conteúdo:

 
```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>API-projeto-jenkins</title>
  <style>
    body {
      font-family: sans-serif;
      text-align: center;
      padding: 20px;
      background-color: #f0f0f0;
    }
    button {
      margin: 10px;
      padding: 10px 20px;
      font-size: 16px;
      cursor: pointer;
    }
    img {
      max-width: 300px;
      margin-top: 20px;
      border-radius: 8px;
    }
  </style>
</head>
<body>
  <h1>🚀 API-projeto-jenkins</h1>

  <button onclick="getColor()">🎨 Mudar Cor</button>
  <button onclick="getCat()">🐱 Gato</button>
  <button onclick="getRandomPhoto()">🖼️ Foto Aleatória</button>
  <button onclick="getTime()">🕒 Hora Atual</button>
  <button onclick="getScare()">😱 Susto</button>
  <button onclick="getLookalike()">👯 Parecido com Você</button>

  <div id="output"></div>

  <script>
    const API = "http://localhost:8001";

    async function getColor() {
      const res = await fetch(`${API}/color`);
      const data = await res.json();
      document.body.style.backgroundColor = data.color;
    }

    async function getCat() {
      const res = await fetch(`${API}/cat`);
      const data = await res.json();
      showImage(data.cat_image_url);
    }

    async function getRandomPhoto() {
      const res = await fetch(`${API}/random-photo`);
      const data = await res.json();
      showImage(data.random_photo_url);
    }

    async function getTime() {
      const res = await fetch(`${API}/time`);
      const data = await res.json();
      document.getElementById("output").innerHTML = `<h2>${data.current_time}</h2>`;
    }

    async function getScare() {
      const res = await fetch(`${API}/scare`);
      const data = await res.json();
      showImage(data.scare_image_url);
    }

    async function getLookalike() {
      const res = await fetch(`${API}/lookalike`);
      const data = await res.json();
      showImage(data.lookalike_image_url);
    }

    function showImage(url) {
      document.getElementById("output").innerHTML = `<img src="${url}" alt="Imagem">`;
    }
  </script>
</body>
</html>
```

  

### Dockerfile do frontend:

```dockerfile
FROM nginx:alpine

COPY ./index.html /usr/share/nginx/html/index.html
```

---
### Docker Compose para testar localmente
Crie o arquivo `compose.yaml` com:
```yaml
services:
  frontend:
    build:
      context: ./frontend
    ports:
     - "3000:80"
    depends_on:
      - backend
  backend:
    build:
      context: ./backend
    ports:
     - "8000:8000"
```

  

---

## Kubernetes (pasta `k8s`)
#### Deployment e Service backend - `deployment-backend.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: desafio-jenkins-backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: desafio-jenkins-backend
  template:
    metadata:
      labels:
        app: desafio-jenkins-backend
    spec:
      containers:
      - name: desafio-jenkins-backend
        image: rafaelldiass/pipedesafiojenkins:backend-{{tag}}
        ports:
        - containerPort: 8000
---
apiVersion: v1
kind: Service
metadata:
  name: desafio-jenkins-service
spec:
  selector:
    app: desafio-jenkins-backend
  ports:
  - port: 80
    targetPort: 8000
    nodePort: 30001
  type: NodePort
```

#### Deployment e Service frontend - `deployment-frontend.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: desafio-jenkins-frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: desafio-jenkins-frontend
  template:
    metadata:
      labels:
        app: desafio-jenkins-frontend
    spec:
      containers:
      - name: desafio-jenkins-frontend
        image: rafaelldiass/pipedesafiojenkins:frontend-{{tag}}
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: desafio-jenkins-frontend-service
spec:
  selector:
    app: desafio-jenkins-frontend
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30002
  type: NodePort

```
---

## Fase 3: Jenkins

1. Crie uma nova tarefa no Jenkins,

selecione **Pipeline**.

2. Em **Triggers**, selecione **GitHub hook trigger for GITScm polling**.

3. Em **Pipeline**:
* Em **Definition**, selecione **Pipeline script from SCM**.

* Em **SCM**, selecione **Git**.

* Insira seu repositório GitHub.

* Na **Branch**, coloque `*/dev` (já que estamos trabalhando na dev).
* 
4. Crie um arquivo `Jenkinsfile` na raiz do seu repositório com o seguinte conteúdo:

```groovy
pipeline{
    agent any
    post {
        always {
            // Chuck Norris aparece em todos os builds
            chuckNorris()
        }
       
        success {
            echo '🚀 Deploy realizado com sucesso!'
            echo '💪 Chuck Norris aprova seu pipeline DevSecOps!'
            echo "✅ Imagem jamalshadowdev/meuapp:${env.BUILD_ID} deployada no Kubernetes"
        }
       
        failure {
            echo '❌ Build falhou, mas Chuck Norris nunca desiste!'
            echo '🔍 Chuck Norris está investigando o problema...'
            echo '💡 Verifique: Docker build, DockerHub push ou Kubernetes deploy'
        }
       
        unstable {
            echo '⚠️ Build instável - Chuck Norris está monitorando'
        }
    }
    stages{
        stage('Docker build backend') {
            steps{
                script{
                    backend = docker.build("rafaelldiass/pipedesafiojenkins:backend-${env.BUILD_ID}", '--no-cache -f ./backend/Dockerfile ./backend' )
                }
            }
        }
         stage('Docker build frontend') {
            steps{
                script{
                    frontend = docker.build("rafaelldiass/pipedesafiojenkins:frontend-${env.BUILD_ID}", '--no-cache -f ./frontend/Dockerfile ./frontend' )
                }
            }
        }
            stage('Push Image') {
            steps{
                script{
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub'){
                        backend.push('latest-backend')
                        backend.push("backend-${env.BUILD_ID}")
                        frontend.push('latest-frontend')
                        frontend.push("frontend-${env.BUILD_ID}")
                    }
                
                }
            }
        }

                    stage('Deploy Kubernetes') {
                        environment {
                            tag_version =  "${env.BUILD_ID}"
                        }
                steps{
                    withKubeConfig([credentialsId: 'kubeconfig']){
                        
                        //backend
                        sh 'sed -i "s/{{tag}}/$tag_version/g" ./k8s/deployment-backend.yaml'
                        sh "kubectl apply -f k8s/deployment-backend.yaml"
                        
                        //frontend

                        sh 'sed -i "s/{{tag}}/$tag_version/g" ./k8s/deployment-frontend.yaml'
                        sh "kubectl apply -f k8s/deployment-frontend.yaml"
                    }
                    
                }
        }
    }
}
```
### Configurando Webhook do GitHub
Para que a pipeline seja atualizada automaticamente após commits:

1. Instale o [ngrok](https://ngrok.com/) em sua máquina local para expor o Jenkins à internet.

2. Rode:

```
ngrok  http  8080
```

3. Copie a URL gerada pelo ngrok.

4. No seu repositório do GitHub, vá em **Settings > Webhooks**.

5. Clique em **Add webhook**.

* Em **Payload URL**, insira a URL do ngrok + `/github-webhook/`. Exemplo:
```
https://abc123.ngrok.io/github-webhook/
```

* Em **Content type**, selecione `application/json`.

* Em **Which events would you like to trigger this webhook?**, selecione somente **Push events**.

---
## Testes da aplicação e exposição dos serviços no ambiente local

  

### Expondo os serviços localmente com kubectl port-forward

  Para testar a aplicação localmente, execute os comandos abaixo para expor os serviços do Kubernetes nas portas locais:

```bash

# Expõe o backend na porta local 8001

kubectl  port-forward  svc/desafio-jenkins-service  8001:80

```
```bash

# Expõe o frontend na porta local 8000

kubectl  port-forward  svc/desafio-jenkins-frontend-service  8000:80
```
---

### Acessando a aplicação

* Backend: [http://localhost:8001](http://localhost:8001)

* Frontend: [http://localhost:8000](http://localhost:8000)

No frontend, os botões da interface irão consumir as APIs do backend exposto na porta 8001, permitindo testar todas as funcionalidades implementadas.

---