# Stable Diffusion Notes

***These instructions contain Linux shell scripts, (not suitable for Windows)***

## Running Automatic1111 and ComfyUI together sharing Python environment and models

### How to use Automatic1111 Python virtual environment (venv) and requirements with ComfyUI.

Create a shell script in the ComfyUI root directory to activate Automatic1111 venv

Example of `/ComfyUI/start.sh` script
```terminal
#!/bin/sh

# Modify paths as required.
PYTHON_ENV=/home/username/stable-diffusion-webui/venv/bin
COMFYUI_PATH=/home/username/ComfyUI

cd $COMFYUI_PATH
# Update ComfyUI to latest source release
git pull

# Activate venv#!/bin/sh

PYTHON_ENV=/home/username/stable-diffusion-webui/venv/bin
COMFYUI_PATH=/home/username/ComfyUI

cd $COMFYUI_PATH
git pull

. $PYTHON_ENV/activate
python3 $COMFYUI_PATH/main.py --listen

. $PYTHON_ENV/activate

# Lauch comfyui --listen makes network accessable
python3 $COMFYUI_PATH/main.py --listen
```

### How to use existing Automatic1111 models in ComfyUI

Edit `extra_model_paths.yaml` located in the ComfyUI root folder.

Example of extra_model_paths.yaml contents:
```terminal
a111:
    base_path: /home/username/stable-diffusion-webui/

    checkpoints: models/Stable-diffusion
    configs: models/Stable-diffusion
    vae: models/VAE
    loras: |
         models/Lora
         models/LyCORIS
    upscale_models: |
                  models/ESRGAN
                  models/RealESRGAN
                  models/SwinIR
    embeddings: embeddings
    hypernetworks: models/hypernetworks
    controlnet: models/ControlNet
```

### How to run Automatic1111 and ComfyUI as a linux systemd service

Below is a shell script to create services for both Automatic1111 and ComfyUI.

**Make sure to make changes to `username` and also paths to application locations in the Service One and Service Two section**



```terminal
#!/bin/sh

############################################
# Debian Linux configure as service script #
############################################


# Function for creating new systemd service
write_service_config() {
	# Check if the service already exists
	echo "Checking for existing ${service_name} service..."
    if systemctl --quiet is-enabled ${service_name} 2>/dev/null
    then
        echo "Service ${service_name} already exists. Skipping..."
        return
    fi
    echo "Creating service for ${service_name}..."
	sudo touch "/var/log/${service_name}.log"
	sudo tee "/etc/systemd/system/${service_name}.service" 1>/dev/null <<EOF
[Unit]
Description=${service_desc}
After=network.target

[Service]
User=${service_user}
Group=${service_group}
WorkingDirectory=${working_dir}
ExecStart=${exec_start}
Restart=always
RestartSec=3
StandardOutput=file:/var/log/${service_name}.log
StandardError=file:/var/log/${service_name}.log

[Install]
WantedBy=multi-user.target
EOF

	echo "Enabling ${service_name} service and reloading service daemon..."
	sudo systemctl enable ${service_name}
	sudo systemctl daemon-reload

	echo "Checking status of ${service_name} service..."
	sudo systemctl status ${service_name}
}

# Service One
# Editing these lines service user, groups and file paths

service_name="automatic1111"
service_desc="Automatic 1111 Stable Diffusion"
service_user="username"
service_group="username"
working_dir="/home/username/stable-diffusion-webui"
exec_start="/home/username/stable-diffusion-webui/webui.sh"

write_service_config

# Service Two
# Editing these lines service user, groups and file paths

service_name="comfyui"
service_desc="ComfyUI Stable Diffusion"
service_user="username"
service_group="username"
working_dir="/home/username/ComfyUI"
exec_start="/home/username/ComfyUI/start.sh"

write_service_config
```





