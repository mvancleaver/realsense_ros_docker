# realsense_ros_docker
Realsense ROS2 Docker Container Example 

# Usage

1) Install Docker and Enable NVIDIA Container Runtime 

```
########################
######## Reinstall Docker
########################
# Remove Existing Docker
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove -y $pkg; done

# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install -y docker docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin nvidia-docker2

sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker

## Edit /etc/docker/daemon.json
# {
#     "runtimes": {
#         "nvidia": {
#             "path": "nvidia-container-runtime",
#             "runtimeArgs": []
#         }
#     },
#     "default-runtime": "nvidia"
# }
```

2) Build the Docker Image 

```
cd realsense_ros_docker/docker/cookbook
docker compose build rs_driver
```

2) Run the Docker Image 

```
cd realsense_ros_docker/docker/cookbook
docker compose run rs_driver
```

# Modify the Container Image 

The Container is build using the Installation Instructions in [`realsense_ros_docker/docker/Dockerfile.realsense`](https://github.com/mvancleaver/realsense_ros_docker/blob/main/docker/Dockerfile.realsense)

# Modify the Container Runtime 

The Runtime Parameters can be configures by modifying [`realsense_ros_docker/docker/cookbook/docker-compose.yaml`](https://github.com/mvancleaver/realsense_ros_docker/blob/main/docker/cookbook/docker-compose.yaml)

```
services:
  rs_driver: # Realsense Container Service
    image: mvancleaver/realsense_ros:foxy # Name of the Docker Container 
    build: # Build Settings 
      context: ../../ # Context Directory for Build Files 
      dockerfile: ./docker/Dockerfile.realsense # Location of the Dockerfile in the Build Context 
    network_mode: host # Use the Host Network/IP Address 
    volumes: # Host Volumes linked to the Container during Runtime 
      - /dev:/dev
      - ../../../config/params:/config
    privileged: true # Allow Connection to USB/Dialout 
    restart: unless-stopped # Restart the Container until it is explicitly stopped  
    command: ros2 launch realsense2_camera rs_launch.py camera_name:='rs_'s unite_imu_method:=2 align_depth.enable:=True enable_gyro:=true enable_accel:=true pointcloud.enable:=true pointcloud.stream_filter:=0 allow_no_texture_point:=true pointcloud.ordered_pc:=true pointcloud.stream_index_filter:=1 # Command to Run when the container starts 
```
