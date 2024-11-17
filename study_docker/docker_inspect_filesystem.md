# Namespaces

## Understand Docker from Namespaces Perspective

To better understand the filesystem organization inside a Docker image, you can inspect the image layers, metadata, and contents. Docker images are built in layers, and each layer contributes to the final filesystem.

Here’s a step-by-step guide to inspecting a Docker image:

---

### 0. **Install Docker**
- [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)

### 1. **Pull the Image**
Pull the Docker image you want to inspect. For example:

```bash
docker pull ubuntu:latest
```

---

### 2. **List Docker Images**
Verify that the image is available locally:

```bash
docker images
```

---

### 3. **Inspect the Metadata**
Use the `docker inspect` command to view the image metadata:

```bash
docker inspect ubuntu:latest
```

Key information to look for:
- **"RootFS":** Shows the layers that make up the image.
- **"Config":** Contains information about environment variables, working directories, etc.

---

### 4. **Save the Image as a Tar File**
Export the image to a tar file for filesystem inspection:

```bash
docker save ubuntu:latest -o ubuntu-latest.tar
```

---

### 5. **Extract the Image Tar File**
Extract the `.tar` file to access the image layers:

```bash
mkdir ubuntu-filesystem
tar -xvf ubuntu-latest.tar -C ubuntu-filesystem
```

This will create a directory with the following:
- **`manifest.json`**: Contains metadata about the image layers.
- **Layer tarballs**: Each tarball corresponds to an image layer (e.g., `sha256:<layer-id>.tar`).
- **`repositories`**: Maps image names and tags to layers.

---

### 6. **Extract and Inspect Layers**
Each layer contributes to the image's filesystem. Extract the layer tar files to view their contents:

```bash
cd ubuntu-filesystem
mkdir layer1 layer2
tar -xvf <layer1-tar>.tar -C layer1
tar -xvf <layer2-tar>.tar -C layer2
```
You can now inspect the files and directories in `layer1`, `layer2`, etc.
---
#### Lab: 
- Inspect manifest.json. In our case it appears ubuntu:latest image only contains a single layer.
- Extract that layer taking tar file id from manifest.json.
```bash
❯ tar -xvf blobs/sha256/a46a5fb872b554648d9d0262f302b2c1ded46eeb1ef4dc727ecc5274605937af -C layer1
```

- Inspect layer1 folder


### 7. **Run a Container and Inspect Live Filesystem** and compare it with layer1 folder structure
To better understand how the layers come together, run a container from the image:

```bash
docker run --interactive --tty --name ubuntu_latest ubuntu:latest bash
```

Inside the container, explore the filesystem structure:

```bash
root@6c04ae6e2118:/# ls -latr /
```

### 8.a. [Optional] **Add a Java runtime layer** and inspect again. 

#### - Create a Docker file

```bash
# Start with the base image
FROM ubuntu:latest

# Add Python 3.12 layer
RUN apt-get update && \
    apt-get install -y software-properties-common && \
    add-apt-repository -y ppa:deadsnakes/ppa && \
    apt-get update && \
    apt-get install -y python3.12 python3.12-distutils && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Add Java Runtime layer
RUN apt-get update && \
    apt-get install -y openjdk-17-jre && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

#### - Build an image
```bash
docker build -t ubuntu-with-python-java .
```

#### - Extract the image and inspect
- Save and extract Docker image
```bash
docker save ubuntu-with-python-java:latest -o ubuntu-python-java-filesystem.tar
mkdir ubuntu-python-java-filesystem
tar -xvf ubuntu-python-java-filesystem.tar -C ubuntu-python-java-filesystem
```

- Inspect filesystem
    - First inspect manifest.json
    - There should be 3 layers. Look closely "Layers" object in manifest.json file
    - Extract layer1
```bash
mkdir layer1
tar -xvf blobs/sha256/a46a5fb872b554648d9d0262f302b2c1ded46eeb1ef4dc727ecc5274605937af -C layer1

mkdir layer2
tar -xvf blobs/sha256/4fdb67b2dc9373a8b64fcade49373eb29d24080cbe38af2f51fafa1d40bcfc70 -C layer2

mkdir layer3
tar -xvf blobs/sha256/0e6583b7531959c1681331c84ef8422483ada0f80c765b562dea3845a9909071 -C layer3

```

#### - Run the image as a container
#### - Inspect live container filesystem and compare.

### 8.b. [Optional] **Use Tools Like `dive`**
Tools like **`dive`** make it easier to explore image layers and understand the filesystem:

#### Install `dive`:
```bash
sudo apt install dive
```

#### Analyze the Image:
```bash
dive ubuntu:latest
```

This interactive tool allows you to:
- View image layers.
- Analyze layer contributions to the final image.
- Check for unnecessary files or potential optimizations.

---

### 9. **Understand Layer Contributions**
Each layer contains:
- **Base OS Filesystem**: The starting point (e.g., Ubuntu).
- **Application Files**: Added during the `COPY` or `ADD` instructions in the Dockerfile.
- **Configuration Changes**: Made via `RUN`, `ENV`, or similar instructions.

---

### 10. **Example: Inspecting Ubuntu Layers**
- **Base Layer**: Contains system directories like `/bin`, `/usr`, `/lib`.
- **Additional Layers**: Add user-specific configurations or application files.
- **Final Layer**: Includes changes from the last `CMD` or `ENTRYPOINT` instruction.

---

By following these steps, you can explore Docker images and gain a detailed understanding of how the filesystem is organized inside them. Tools like `dive` and manual inspection of layers can provide deeper insights into image optimization and layer hierarchy.


