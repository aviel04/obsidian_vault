## Containerizing NVIDIA Parakeet with Docker/Podman Compose

Here's a comprehensive guide to deploying an NVIDIA Parakeet application using Docker Compose (and Podman Compose). This setup will include GPU support for accelerated inference.

### 1. Dockerfile Setup

- Use an NVIDIA CUDA base image.
    
- Install system dependencies (Python, pip, etc.).
    
- Install the NVIDIA NeMo toolkit (or Riva, depending on your workflow).
    
- Download the pre-trained Parakeet model.
    
- Copy your application code into the container.
    
- Set the command to run your application.
    

```dockerfile
FROM nvidia/cuda:12.3.2-cudnn8-devel-ubuntu22.04

WORKDIR /app

RUN apt-get update && apt-get install -y --no-install-recommends \
    python3 \
    python3-pip \
    git \
    curl \
    libsndfile1 \
    && rm -rf /var/lib/apt/lists/*

RUN pip install --upgrade pip setuptools wheel
RUN pip install nemo_toolkit['all']

RUN nemo_download_pretrained(model_name="stt_en_conformer_ctc_large")

COPY . .

CMD ["python3", "your_inference_script.py"]
```

### 2. Docker Compose Configuration

- Define the `parakeet-app` service.
    
- Specify the Docker image (built from the `Dockerfile`).
    
- Enable NVIDIA GPU support.
    
- Configure port mappings and volumes.
    

```yaml
version: '3.8'

services:
  parakeet-app:
    image: parakeet-nemo
    runtime: nvidia # For Docker
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]
    ports:
      - "8080:8080"
    volumes:
      - ./audio_data:/app/audio_data
```

### 3. Podman Compose Considerations

- `docker-compose.yaml` is generally compatible with `podman-compose`.
    
- GPU support in Podman may require different configuration:
    
    - Remove `runtime: nvidia`.
        
    - Use device mappings in `deploy.resources.reservations`:
        

```yaml
deploy:
  resources:
    reservations:
      devices:
        - host_path: /dev/nvidia0
          container_path: /dev/nvidia0
          permissions: rwm
        - host_path: /dev/nvidiactl
          container_path: /dev/nvidiactl
          permissions: rwm
        - host_path: /dev/nvidia-uvm
          container_path: /dev/nvidia-uvm
          permissions: rwm
        - host_path: /dev/nvidia-uvm-mgmt
          container_path: /dev/nvidia-uvm-mgmt
          permissions: rwm
```

### 4. Running the Application

- Build the image:
    
    - `docker build -t parakeet-nemo .`
        
    - `podman build -t parakeet-nemo .`
        
- Start the services:
    
    - `docker-compose up --detach`
        
    - `podman-compose up --detach`