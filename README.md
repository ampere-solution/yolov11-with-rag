# yolov11-with-rag - How-to
### Getting Started
1. **Download** the Ampere optimized 'Llama-3.2-3B-Instruct-Q4_K_4.gguf' model from [Hugging Face](https://huggingface.co).
2. **Place** the model inside the `models/` directory.
3. **Place** the videos inside the `videos/` directory.
4. **Run** the setup script:
   ```bash
   ./start_app.sh

The script will pull the demo docker image from docker hub, setup the environments neccessary for this demo.
**Open** the demo at http://< your_ip_address >:5070
