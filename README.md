# YOLOv11 Object Detection and Tracking
## User Interface
This is a real-time video analytics platform showcasing AI on Ampere CPUs.  It combines four capabilities in one dashboard:
- Object Detection & Tracking - YOLOv11 inference with Bytetrack assigns persistent IDs to objects across frames, drawn live on an MJPEG video feed.
- Local LLM Q&A with RAG - A GGUF model (via llama-cpp-python) answer natural language questions about what happening in the video, with question aware section selection so the LLM only sees relevant data.
- Observability - Time series metrics stream to a Flask + SOcket.IO web UI and a companion Grafana dashboard.
<img width="2091" height="1094" alt="image-20260504-170501.png" src="image-20260504-170501.png" />

## Architectural Diagram
This demo is a Flask + Socket.IO single process app that run YOLOv11 + ByteTrack inference on video frames, streams annotated MJPEG and live metrics to a vanilla-JS dashboard - all coordinated by background threads optimized for Ampere processors. A local GGUF LLM (via llama-cpp-python) answers questions about the scene using RAG context built from the live tracking timeline. 

<img width="2091" height="1094" alt="image-20260504-173532.png" src="image-20260504-173532.png" />

## What This Demo Shows
The demo showcases real-time video analytics running entirely on Ampere processors - no GPU required.   Specially, it demonstrates:
- **Live object detection and tracking** - YOLOv11 + ByteTrack assigns persistent IDs to people, vehicles, and other objects as they move through the scene.
- **Deployed LLM Q&A** - A local GGUF model answer natural language questions using RAG context built from the tracking timeline.
- **Production-grade observability** - Live FPS, tracked, and visible objects,  CPU and Memory gauges

## Target Audience
The demo is built to speak to several groups:
- Enterprise buyers - IT decision makers evaluating Ampere servers for AI workloads.  Who need proof that Ampere CPUs can run modern vision + LLM stacks without GPUs.
- Cloud & Edge infrastructure architectures - Engineers designing video analytics, smart-retail, smart-city, or industrial monitoring deployments where GPU cost, power, or supply constraints matter.
- Solution partners & ISVs - Companies building video analytics products who could port or co-develop on Ampere, using this as a reference architecture (detection + tracking + RAG + observability in one container)
- Trade show & briefing center visitors - Non technical executives and analysts who need a visually compelling, self running demo that tells the “full AI stack on Ampere“ story at a glance.

## Key Message  - What are we trying to convince of?
**Core message**

Ampere processors can run the full modern AI stack - vision + tracking + LLM + observability in production, today, without GPUs.
- CPU only inference is real, not a compromise 
   - YOLOv11 + ByteTrack runs at ~30fps on Ampere
   - Quantized GGUF LLMs deliver useful Q&A latency
   - Convince:  You don’t need a GPU budget to deploy video AI.
- The whole stack fits in one container
   - Detection, tracking, LLM, RAG, metrics, and UI in a single Flask process
   - Convince:  Edge and on-prem deployment is operationally simple
- Private, local AI is viable
   - LLM runs locally
   - No data leaves the box
   - Convince:  You can have RAG-powered Q&A without sending videos or telemetry to the cloud.
- Production ready
   - Convince:  this is a reference architecture, not a science project.
- Total cost of ownership advantages 
   - Ampere processors - lower power, lower cost, broader availability than GPUs
   - Convince:  switching to Ampere reduce TCO without sacrificing capability
- Developer friendly and extensible
   - Standard python stack (Flask, Socket.IO, Ultralytics, llama-cpp-python)
   - Convince:  yourour team can build on this.

Overall, Ampere processors are a credible, cost effective, production grade platform for the full AI stack - vision and LLMs together.  

## Proof Points - How does It Show This?
- CPU-only inference is production viable
   - YOLOv11 runs at real-time FPS on CPU: Stats bar display live FPS
   - No GPU dependency: docker stats shows zero GPU usage;  Dockerfile uses Ampere base image with no CUDA
- Local LLM delivers usable Q&A latency
   - Fast time to first token: Q&A response shows TTFT in ms next to every answer
   - Competitive throughput: Same bar show tokens/sec
   - Useful answer quality: Ask layered questions - answers are accurate
- Private, local AI
   - Data never leaves the box: visibly assembled from local timeline
   - GGUF models are interchangeable: Switch to different model - no code change
- TCO advantage over GPU pipelines
   - Power efficiency: Grafana GPU panel show modest utilization for full pipeline
   - Core scaling: Pinned to 40 Ampere cores.
   - Single server replaces GPU rig: Compare deployment footprint verbally - Ampere CPU only vs. CPU host + GPU
 
## Running the Demo
**Recommended Resources**
Step-by-step guide to run the Object Detection and Tracking with RAG demo in Docker.
- Minimum: 40 cores. Recommended: 40+ cores
- Minimum: 32GB RAM. Recommended: 64+ GB RAM
- Minimum: 20GB disk space. Recommended: 100+ GB (multiple models, large models)

**Software Stack**
- Base Image: amperecomputingai/llama.cpp:3.4.2-ampereone
- Python: 3.3
- Flask: 3.1.0
- Flask-SocketIO: 5.4.1
- Chart.js: 4.4.1
- OpenCV (headless): =4.9.0
- llama-cpp-python: 0.3.16
- Grafana: 11.4.0
- Docker Compose: v2.0+

**Demo Deployment**
- Install docker 
- Git clone from the repo:  GitHub - ampere-solution/yolov11-with-rag 
- Download the Ampere optimized 'Llama-3.2-3B-Instruct-Q4_K_4.gguf' model
- Place the model inside the models/ directory.
- Place the videos inside the videos/ directory.
- docker-compose.yaml:
```yaml
services:
  app:
    cpuset: "0-39"
    image: tinguyen2024/visual-intellegent-systgem:v1.2
    #build:
    #  context: .
    #  dockerfile: Dockerfile
    ports:
      - "5070:5070"
    volumes:
      - ./videos:/app/videos:ro
      - ./models:/app/models:ro
      - tracking-data:/data
    environment:
      - DB_PATH=/data/tracking.db
      - VIDEOS_DIR=/app/videos
      - MODELS_DIR=/app/models
      - LLM_THREADS=32
      - LLM_THREADS_BATCH=32
      - LLM_CTX=100000
      - LLM_BATCH=512
      - AUTO_START_VIDEO=driving-ny.mp4
      - AUTO_LOAD_LLM=Llama-3.2-3B-Instruct-Q4_K_4.gguf
    restart: unless-stopped

  grafana:
    image: grafana/grafana:11.4.0
    ports:
      - "3000:3000"
    volumes:
      - grafana-data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning:ro
      - ./grafana/dashboards:/var/lib/grafana/dashboards:ro
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_AUTH_ANONYMOUS_ENABLED=true
      - GF_AUTH_ANONYMOUS_ORG_ROLE=Viewer
      - GF_INSTALL_PLUGINS=simpod-json-datasource
      - GF_DASHBOARDS_DEFAULT_HOME_DASHBOARD_PATH=/var/lib/grafana/dashboards/overview.json
    depends_on:
      - app
    restart: unless-stopped

volumes:
  tracking-data:
  grafana-data:
```
- Run 'start_app.sh'. The script will pull the demo docker image from docker hub, setup the environments neccessary for this demo.
- Open the demo at http://< your_ip_address >:5070

**Demo Talking Points**
- CPU-only, full AI stack - YOLOv11 detection, ByteTrack multi-object tracking, and a local GGUF LLM all running together at ~30fps on Ampere processors. 
- Persistent multi-object tracking - Each object gets a stable track ID across frames
- Live-data RAG - The LLM answers questions about what’s happening.
- Question-aware context selection - The system inspects each question and sends only the relevant RAG sections
- Radical transparency - returns the exact text the LLM sees
- Production observability build-in

**Stop the Demo**
- Graceful stop
```bash
# stop_app.sh
$ docker compose stop
```
- Remove the demo
```bash
$ docker compose down
```

