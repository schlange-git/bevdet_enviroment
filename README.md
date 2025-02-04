# bevdet_enviroment
The current version still encounters compatibility issues even when using the official Docker configuration for the local environment. Below is a tutorial for setting up the environment using Docker on Ubuntu 20.04.

# Bilibili Teaching Videos

- [Video 1](https://www.bilibili.com/video/BV1s54y1n7Ev/?spm_id_from=333.337.search-card.all.click&vd_source=4b8e5a437540b388abe0e0e23e80b583)
- [Video 2](https://www.bilibili.com/video/BV1Dm4y147eM/?spm_id_from=333.337.search-card.all.click&vd_source=4b8e5a437540b388abe0e0e23e80b583)

---

## Docker Logic

### **Image Setup**
Create the following **bash script** and execute it **on the local environment (not inside Docker)**:

```bash
# Add NVIDIA repository
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

# Update packages and install the NVIDIA Container Toolkit
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure
sudo systemctl restart docker
```

### **Entering the Docker Environment**
Run the following command to start the container:
```bash
docker run --gpus all -it -v /home/shi/Desktop/models/BEVDet:/workspace mmdet bash
```
- **Modify `/home/shi/Desktop/models/BEVDet`** to your actual **local directory**.
- **`mmdet`** is the **container name**.

In **VS Code**, you can **stop** or **connect to the shell** via the **Containers** panel by right-clicking.

To test whether the script worked correctly, run:
```bash
nvidia-smi
```

---

## **Step 2: Running the BEVDet Installation**
Refer to the **official tutorial**:  
[BEVDet GitHub Repository](https://github.com/HuangJunJie2017/BEVDet/tree/dev3.0)

Clone the repository:
```bash
git clone https://github.com/HuangJunJie2017/BEVDet.git
cd BEVDet
pip install -v -e .
```

Before proceeding, enter your **mounted directory**:
```bash
cd /workspace
```

---

## **Fixing Docker Installation and Dependencies**
Even if everything is executed correctly, Docker may still fail to install some dependencies. The following changes may be required:

### **Updating Conda**
```bash
conda update --force conda
```

### **Reinstalling `numba` and `llvmlite`**
```bash
conda install --force-reinstall numba llvmlite
```

### **Downgrading `numpy` (must be below version 1.20)**
```bash
conda install numpy==1.19.5
```

### **Fixing Pandas Import Errors**
If `pandas` raises an import error, install it using Conda and downgrade:
```bash
conda install pandas
conda install pandas==1.3.3
```
Neither version alone may work; install both sequentially.

---

## **Fixing `spconv` Import Errors**
If you encounter:
```
ModuleNotFoundError: No module named 'spconv'
```
Check your CUDA version:
```bash
nvcc --version
```
Then install the correct `spconv` version (example for CUDA 11.7):
```bash
pip install spconv-cu117
```
Modify `cu117` according to your CUDA version.

---

## **Dataset Preparation & Training**
Some scripts require configuration files to work properly.

### **1️⃣ Creating the dataset**
```bash
python tools/create_data_bevdet.py
```

### **2️⃣ Running training**
```bash
python tools/train.py /workspace/configs/bevdet/bevdet-r50-4d-cbgs.py
```
Ensure that the **config file exists** and is properly referenced.

---

## **Final Notes**
- This guide ensures that **Docker is correctly set up** for BEVDet.
- If issues arise, check logs and try **reinstalling dependencies** as needed.
- Ensure your **mount directory is correctly configured** before training.

🚀 Happy Training!
