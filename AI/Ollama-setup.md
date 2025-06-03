# Setting stuff up

```bash
sudo apt install podman # use to containarize ollama server

curl -fsSL https://ollama.com/install.sh | sh # the ollama cli client to talk with the server

export OLLAMA_HOST="127.0.0.1:11434" # server host address
nvidia-smi # get the uid of the prefered GPU
export CUDA_VISIBLE_DEVICES=0 # set the uid of the GPU
```


## Ollama Models
```bash
podman run -d --rm -p 127.0.0.1:11434:11434 --gpus all --name ollama_server ollama/ollama

ollama pull wizardcoder:15b-python-q4_K_M
```
- `q4_km` is the amount of quantization larger numbers are heavier on the GPU
- `--gpus all` lets the Ollama container access all the GPUS in the system 


---
# Simple installation of [[ollama]]

- install docker compose
- install ollama cli on wsl
- pull ollama/ollama 
- run it using the `--gpus all` flag
- pull models

install Nvidia tools for containers ( Some research in the internet might be necessary )
```bash
sudo apt install nvidia-smi
sudo apt install nvidia-container-toolkit
sudo nvidia-container-toolkit -t docker --set-as-default-runtime
sudo apt install nvidia-container-runtime
```

the ollama server container with GPU access
```bash
docker run --rm --gpus all -p 11434:11434 -v ollama:/root/.ollama -d ollama/ollama
```

----
## After running ad-hoc container successfully:
we can now use a docker compose solution:
```yaml 
services:
  ollama_server:
    image: ollama/ollama # The Docker image 
    container_name: ollama_server # Sets a custom name 
    ports:
      - "11434:11434" # Maps port 11434 on the host to port 11434 
                      # Removed "127.0.0.1:" prefix to allow access  
                      # if you only want localhost access, add "127.0.0.1:" back.
    volumes:
      - ollama_data:/root/.ollama # Mounts the named volume 'ollama_data' 
                                  # Ensures your downloaded models persist 
    deploy: # Configuration for deploying the service, specifically for GPU access
      resources:
        reservations:
          devices:
            - driver: nvidia # Specifies the NVIDIA driver
              count: all     # Exposes all available NVIDIA GPUs 
              capabilities: [gpu, utility, compute] # Requests necessary GPU 
    restart: unless-stopped # Configures the container to restart auto
volumes:
  ollama_data: # Defines the named volume 'ollama_data'. 
```
### Errors:
**No GPU in the container run time**
check if docker container has access to GPU:
```bash
docker run --rm --gpus all nvidia/cuda:12.1.1-base-ubuntu22.04 nvidia-smi
```



