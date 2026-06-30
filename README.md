# YOLOv11 Object Detection and Tracking
## User Interface
This is a real-time video analytics platform showcasing AI on Ampere CPUs.  It combines four capabilities in one dashboard:
- Object Detection & Tracking - YOLOv11 inference with Bytetrack assigns persistent IDs to objects across frames, drawn live on an MJPEG video feed.
- Local LLM Q&A with RAG - A GGUF model (via llama-cpp-python) answer natural language questions about what happening in the video, with question aware section selection so the LLM only sees relevant data.
- Observability - Time series metrics stream to a Flask + SOcket.IO web UI and a companion Grafana dashboard.
<img width="2091" height="1094" alt="image-20260504-170501.png" src="image-20260504-170501.png" />

## Architectural Diagram













----------
# yolov11-with-rag - How-to
### Getting Started
1. **Download** the Ampere optimized 'Llama-3.2-3B-Instruct-Q4_K_4.gguf' model from [Hugging Face](https://huggingface.co).
2. **Place** the model inside the `models/` directory.
3. **Place** the videos inside the `videos/` directory.
4. **Run** the setup script:
   ```bash
   ./start_app.sh

**The script** will pull the demo docker image from docker hub, setup the environments neccessary for this demo.

**Open** the demo at http://< your_ip_address >:5070
