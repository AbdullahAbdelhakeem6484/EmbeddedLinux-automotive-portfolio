# Portfolio Setup Guide 🛠️
## Complete Development Environment Setup

> **Comprehensive guide for setting up the development environment for all 4 automotive embedded Linux projects**

## 📋 Prerequisites

### Hardware Requirements
- **Development Host**: Ubuntu 22.04 LTS (minimum 16GB RAM, 200GB free space)
- **Target Hardware**: 
  - Raspberry Pi 4B (8GB) - Projects 1-2
  - BeagleBone Black - Project 3
  - NVIDIA Jetson AGX Xavier - Project 4
- **Additional Hardware**:
  - MicroSD cards (32GB+ Class 10)
  - CAN bus modules (MCP2515)
  - Touch displays (7" and 10")
  - Development accessories (cables, power supplies)

### Software Dependencies
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Yocto Project dependencies
sudo apt install -y \
    gawk wget git diffstat unzip texinfo gcc build-essential \
    chrpath socat cpio python3 python3-pip python3-pexpect \
    xz-utils debianutils iputils-ping python3-git python3-jinja2 \
    libegl1-mesa libsdl1.2-dev python3-subunit mesa-common-dev \
    zstd liblz4-tool file locales libacl1 vim

# Install additional development tools
sudo apt install -y \
    can-utils \
    docker.io \
    docker-compose \
    cmake \
    qtbase5-dev \
    qtdeclarative5-dev \
    git-lfs

# Configure git (replace with your details)
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

## 🚀 Project Setup Workflow

### Universal Setup Script
```bash
#!/bin/bash
# setup_portfolio.sh - Universal portfolio setup script

set -e

PORTFOLIO_ROOT="$HOME/embedded-linux-portfolio"
YOCTO_CACHE="$HOME/yocto-cache"

echo "Setting up Embedded Linux & Yocto Portfolio..."

# Create directory structure
mkdir -p $PORTFOLIO_ROOT
mkdir -p $YOCTO_CACHE/{downloads,sstate-cache}

cd $PORTFOLIO_ROOT

# Clone Yocto Project base
git clone -b kirkstone git://git.yoctoproject.org/poky
cd poky
git clone -b kirkstone git://git.openembedded.org/meta-openembedded
git clone -b kirkstone git://git.yoctoproject.org/meta-raspberrypi
git clone -b kirkstone git://git.yoctoproject.org/meta-qt5

# Set up shared state cache for faster builds
echo "Setting up shared state cache..."
cat << 'EOF' > $PORTFOLIO_ROOT/common-cache.conf
# Shared cache configuration for all projects
DL_DIR = "${YOCTO_CACHE}/downloads"
SSTATE_DIR = "${YOCTO_CACHE}/sstate-cache"
BB_NUMBER_THREADS = "8"
PARALLEL_MAKE = "-j 8"
EOF

echo "Portfolio setup complete!"
echo "Next steps:"
echo "1. cd $PORTFOLIO_ROOT"
echo "2. Choose a project directory (01_beginner_autodisplay, etc.)"
echo "3. Follow the project-specific setup instructions"
```

### Project-Specific Setup

#### Project 1: AutoDisplay-Yocto
```bash
cd $PORTFOLIO_ROOT
mkdir -p projects/01_beginner_autodisplay
cd projects/01_beginner_autodisplay

# Initialize build environment
source ../../poky/oe-init-build-env build-autodisplay

# Configure build
cat ../../../common-cache.conf >> conf/local.conf
echo 'MACHINE = "raspberrypi4-64"' >> conf/local.conf
echo 'DISTRO = "poky"' >> conf/local.conf

# Add layers
bitbake-layers add-layer ../../poky/meta-openembedded/meta-oe
bitbake-layers add-layer ../../poky/meta-openembedded/meta-python
bitbake-layers add-layer ../../poky/meta-raspberrypi
bitbake-layers add-layer ../../poky/meta-qt5
```

#### Project 2: InfuTainment-Platform
```bash
cd $PORTFOLIO_ROOT
mkdir -p projects/02_intermediate_infotainment
cd projects/02_intermediate_infotainment

source ../../poky/oe-init-build-env build-infotainment

# Enhanced configuration for infotainment
cat ../../../common-cache.conf >> conf/local.conf
echo 'MACHINE = "raspberrypi4-64"' >> conf/local.conf
echo 'DISTRO = "infotainment"' >> conf/local.conf
echo 'IMAGE_FEATURES += "ssh-server-openssh package-management"' >> conf/local.conf
echo 'EXTRA_IMAGE_FEATURES += "debug-tweaks"' >> conf/local.conf

# Add multimedia layers
bitbake-layers add-layer ../../poky/meta-openembedded/meta-oe
bitbake-layers add-layer ../../poky/meta-openembedded/meta-python
bitbake-layers add-layer ../../poky/meta-openembedded/meta-multimedia
bitbake-layers add-layer ../../poky/meta-raspberrypi
bitbake-layers add-layer ../../poky/meta-qt5
```

#### Project 3: CAN-RT-Gateway
```bash
cd $PORTFOLIO_ROOT
mkdir -p projects/03_advanced_can_gateway
cd projects/03_advanced_can_gateway

source ../../poky/oe-init-build-env build-can-gateway

# RT kernel configuration
cat ../../../common-cache.conf >> conf/local.conf
echo 'MACHINE = "beaglebone-yocto"' >> conf/local.conf
echo 'PREFERRED_PROVIDER_virtual/kernel = "linux-yocto-rt"' >> conf/local.conf
echo 'KERNEL_FEATURES += "features/latency/latency.scc"' >> conf/local.conf
echo 'IMAGE_INSTALL += "rt-tests can-utils"' >> conf/local.conf

# Add CAN support
echo 'KERNEL_MODULE_AUTOLOAD += "can can-dev can-raw mcp251x"' >> conf/local.conf
```

#### Project 4: AV-Edge-Fusion
```bash
cd $PORTFOLIO_ROOT
mkdir -p projects/04_expert_av_platform
cd projects/04_expert_av_platform

source ../../poky/oe-init-build-env build-av-platform

# AI/ML optimized configuration
cat ../../../common-cache.conf >> conf/local.conf
echo 'MACHINE = "jetson-agx-xavier-devkit"' >> conf/local.conf
echo 'DISTRO = "autonomous-vehicle"' >> conf/local.conf
echo 'IMAGE_INSTALL += "tensorflow-lite pytorch-mobile opencv"' >> conf/local.conf
echo 'CUDA_INSTALL = "1"' >> conf/local.conf
echo 'MACHINE_FEATURES += "cuda gpu"' >> conf/local.conf
```

## 🔧 Development Tools Setup

### CAN Bus Development
```bash
# Install CAN utilities
sudo apt install can-utils

# Set up virtual CAN interface for testing
sudo modprobe vcan
sudo ip link add dev vcan0 type vcan
sudo ip link set up vcan0

# Test CAN communication
cansend vcan0 123#DEADBEEF
candump vcan0
```

### Cross-Compilation Environment
```bash
# Install cross-compilation toolchain
sudo apt install gcc-aarch64-linux-gnu g++-aarch64-linux-gnu

# Set up environment variables
export CROSS_COMPILE=aarch64-linux-gnu-
export CC=${CROSS_COMPILE}gcc
export CXX=${CROSS_COMPILE}g++
export AR=${CROSS_COMPILE}ar
export STRIP=${CROSS_COMPILE}strip
```

### Docker Development Environment
```bash
# Create Docker development container
cat << 'EOF' > Dockerfile.yocto
FROM ubuntu:22.04

# Install Yocto dependencies
RUN apt-get update && apt-get install -y \
    gawk wget git diffstat unzip texinfo gcc build-essential \
    chrpath socat cpio python3 python3-pip python3-pexpect \
    xz-utils debianutils iputils-ping python3-git python3-jinja2 \
    libegl1-mesa libsdl1.2-dev python3-subunit mesa-common-dev \
    zstd liblz4-tool file locales vim

# Create build user
RUN useradd -m -s /bin/bash yocto
USER yocto
WORKDIR /home/yocto

# Set up git
RUN git config --global user.name "Portfolio Builder"
RUN git config --global user.email "builder@example.com"

CMD ["/bin/bash"]
EOF

# Build development container
docker build -t yocto-dev -f Dockerfile.yocto .

# Run development container
docker run -it --rm -v $(pwd):/workspace yocto-dev
```

## 📊 Performance Optimization

### Build Performance
```bash
# Optimize build performance
echo 'BB_NUMBER_THREADS = "$(nproc)"' >> conf/local.conf
echo 'PARALLEL_MAKE = "-j $(nproc)"' >> conf/local.conf

# Use tmpfs for temporary builds (if you have enough RAM)
echo 'TMPDIR = "/tmp/yocto-tmp"' >> conf/local.conf

# Enable build statistics
echo 'INHERIT += "buildstats buildstats-summary"' >> conf/local.conf
```

### Shared State Cache
```bash
# Set up shared state mirror (optional)
echo 'SSTATE_MIRRORS = "file://.* http://sstate.yoctoproject.org/3.1/PATH;downloadfilename=PATH"' >> conf/local.conf

# Enable hash equivalence for faster builds
echo 'BB_HASHSERVE = "auto"' >> conf/local.conf
echo 'BB_SIGNATURE_CONSUME_CACHE = "auto"' >> conf/local.conf
```

## 🧪 Testing and Validation

### Automated Testing Setup
```bash
# Create test infrastructure
mkdir -p tests/{unit,integration,performance}

# Unit testing for custom recipes
cat << 'EOF' > tests/unit/test_recipes.py
#!/usr/bin/env python3
import unittest
import subprocess

class TestRecipes(unittest.TestCase):
    def test_recipe_parsing(self):
        """Test that custom recipes parse correctly"""
        result = subprocess.run(['bitbake', '-p'], capture_output=True)
        self.assertEqual(result.returncode, 0)
        
    def test_custom_layers(self):
        """Test custom layer compatibility"""
        result = subprocess.run(['bitbake-layers', 'show-layers'], capture_output=True)
        self.assertEqual(result.returncode, 0)

if __name__ == '__main__':
    unittest.main()
EOF

# Integration testing
cat << 'EOF' > tests/integration/test_builds.sh
#!/bin/bash
# Integration test for image builds

set -e

PROJECTS=("autodisplay" "infotainment" "can-gateway" "av-platform")

for project in "${PROJECTS[@]}"; do
    echo "Testing $project build..."
    cd projects/*/build-$project
    bitbake -n ${project}-image
    echo "$project build test passed"
done
EOF

chmod +x tests/integration/test_builds.sh
```

## 📈 CI/CD Integration

### GitHub Actions Workflow
```yaml
# .github/workflows/portfolio-ci.yml
name: Portfolio CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test-recipes:
    runs-on: ubuntu-22.04
    steps:
    - uses: actions/checkout@v3
    
    - name: Install dependencies
      run: |
        sudo apt update
        sudo apt install -y gawk wget git diffstat unzip texinfo gcc build-essential
        
    - name: Setup Yocto
      run: |
        git clone -b kirkstone git://git.yoctoproject.org/poky
        source poky/oe-init-build-env
        
    - name: Test recipe parsing
      run: |
        cd build
        bitbake -p
        
    - name: Validate custom layers
      run: |
        cd build
        bitbake-layers show-layers
```

## 🎯 Success Metrics

### Build Performance Targets
- **Initial Build Time**: <4 hours for basic image
- **Incremental Build**: <30 minutes for recipe changes
- **Clean Build**: <2 hours with shared state cache
- **Storage Usage**: <50GB per project workspace

### Development Workflow Metrics
- **Setup Time**: <1 hour for complete environment
- **Project Switch**: <5 minutes between projects
- **Build Success Rate**: >95% for documented configurations
- **Documentation Coverage**: 100% of critical procedures

## 🔧 Troubleshooting Guide

### Common Issues and Solutions

#### Build Failures
```bash
# Clear build cache
rm -rf tmp/

# Reset bitbake cache
bitbake -c cleanall <recipe-name>

# Check disk space
df -h
```

#### Layer Conflicts
```bash
# Check layer priorities
bitbake-layers show-layers

# Validate layer compatibility
bitbake-layers check-layers
```

#### Performance Issues
```bash
# Monitor build resources
htop

# Check I/O performance
iotop

# Optimize parallel builds
echo 'BB_NUMBER_THREADS = "4"' >> conf/local.conf
```

## 📞 Support and Resources

### Community Resources
- **Yocto Project Mailing Lists**: https://lists.yoctoproject.org/
- **Yocto Project Documentation**: https://docs.yoctoproject.org/
- **Stack Overflow**: Tag your questions with `yocto`, `embedded-linux`

### Professional Development
- **Yocto Project Training**: https://www.yoctoproject.org/training/
- **Embedded Linux Conferences**: ELC, ELCE, Embedded World
- **Online Courses**: Linux Foundation, Coursera, Udemy

---

> **Setup Complete**: Your development environment is now ready for building all 4 portfolio projects. Start with Project 1 and progress through the skill levels systematically.

### 🏷️ Setup Tags
`#YoctoSetup` `#EmbeddedLinux` `#DevelopmentEnvironment` `#AutomotiveEmbedded` `#CrossCompilation` `#BuildOptimization` `#CI/CD` `#PortfolioDevelopment` 