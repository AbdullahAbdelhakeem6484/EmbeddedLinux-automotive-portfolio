# AutoDisplay-Yocto Architecture 📐

## System Architecture Overview

### High-Level System Architecture
```mermaid
graph TB
    A[Development Host<br/>Ubuntu 22.04] --> B[Yocto Build System<br/>Kirkstone LTS]
    B --> C[Custom Layers]
    C --> D[meta-autodisplay]
    C --> E[meta-rpi-automotive]
    
    D --> F[AutoDisplay Image]
    E --> F
    
    F --> G[Raspberry Pi 4B<br/>Target Hardware]
    G --> H[7" Touch Display]
    G --> I[Vehicle Simulation]
    
    subgraph "Target System"
        J[Linux Kernel 6.1]
        K[Qt5/QML Runtime]
        L[Wayland Compositor]
        M[Vehicle Services]
    end
    
    G --> J
    J --> K
    K --> L
    L --> M
    M --> H
```

### Yocto Layer Architecture
```mermaid
graph TD
    A[poky - Base Yocto] --> B[Layer Stack]
    
    subgraph "Core Layers"
        C[meta-openembedded]
        D[meta-raspberrypi]
        E[meta-qt5]
    end
    
    subgraph "Custom Layers"
        F[meta-autodisplay<br/>Main automotive layer]
        G[meta-rpi-automotive<br/>BSP customization]
    end
    
    B --> C
    B --> D
    B --> E
    B --> F
    B --> G
    
    F --> H[autodisplay-image.bb<br/>Custom Image Recipe]
    
    subgraph "Recipe Categories"
        I[recipes-automotive/<br/>Display & Vehicle UI]
        J[recipes-core/<br/>Images & Base]
        K[recipes-graphics/<br/>Qt5 & Wayland]
    end
    
    F --> I
    F --> J
    F --> K
```

### Build Process Flow
```mermaid
flowchart TD
    A[Developer] --> B[bitbake autodisplay-image]
    B --> C[Parse Recipes & Layers]
    C --> D[Fetch Sources]
    D --> E[Cross Compile]
    E --> F[Package Creation]
    F --> G[Rootfs Assembly]
    G --> H[Image Generation]
    H --> I[SD Card Image]
    
    subgraph "Build Stages"
        J[do_fetch]
        K[do_configure]
        L[do_compile]
        M[do_install]
        N[do_package]
    end
    
    E --> J
    J --> K
    K --> L
    L --> M
    M --> N
    
    I --> O[Flash to SD Card]
    O --> P[Boot on Raspberry Pi]
```

### Runtime System Architecture
```mermaid
graph TB
    A[Boot Loader<br/>U-Boot] --> B[Linux Kernel 6.1<br/>ARM64]
    B --> C[systemd Init System]
    
    subgraph "System Services"
        D[Wayland Compositor]
        E[Qt5 Platform Plugin]
        F[AutoDisplay Service]
        G[Vehicle Simulator]
    end
    
    C --> D
    C --> E
    C --> F
    C --> G
    
    subgraph "Application Layer"
        H[AutoDisplay QML App]
        I[Touch Input Handler]
        J[Vehicle Data Provider]
    end
    
    F --> H
    E --> I
    G --> J
    
    H --> K[7" Touch Display<br/>1024x600]
    I --> K
    
    subgraph "Hardware Interfaces"
        L[DSI Display Interface]
        M[I2C Touch Controller]
        N[GPIO Status LEDs]
    end
    
    K --> L
    I --> M
    J --> N
```

### Software Component Interaction
```mermaid
sequenceDiagram
    participant U as User
    participant T as Touch Input
    participant Q as Qt5 Application
    participant V as Vehicle Simulator
    participant D as Display
    
    Note over U,D: System Startup
    Q->>D: Initialize Display
    Q->>V: Start Vehicle Simulation
    V->>Q: Send Initial Vehicle Data
    Q->>D: Render Dashboard
    
    Note over U,D: User Interaction
    U->>T: Touch Screen
    T->>Q: Touch Event
    Q->>Q: Process Input
    Q->>D: Update Display
    
    Note over U,D: Periodic Updates
    loop Every 1 second
        V->>Q: Vehicle Data Update
        Q->>Q: Update Dashboard State
        Q->>D: Refresh Display
    end
```

### Data Flow Architecture
```mermaid
graph LR
    A[Vehicle Simulator] --> B[Vehicle Data<br/>JSON Format]
    B --> C[Qt5 Data Model]
    C --> D[QML UI Components]
    D --> E[Rendered Display]
    
    subgraph "Data Types"
        F[Speed: 0-200 km/h]
        G[RPM: 0-8000]
        H[Fuel Level: 0-100%]
        I[Engine Temp: 60-120°C]
        J[Status Indicators]
    end
    
    B --> F
    B --> G
    B --> H
    B --> I
    B --> J
    
    subgraph "Display Elements"
        K[Speed Gauge]
        L[RPM Meter]
        M[Fuel Indicator]
        N[Temperature Gauge]
        O[Warning Lights]
    end
    
    F --> K
    G --> L
    H --> M
    I --> N
    J --> O
```

### Development Workflow
```mermaid
gitgraph
    commit id: "Project Setup"
    branch layer-development
    checkout layer-development
    commit id: "Create meta-autodisplay"
    commit id: "Add vehicle UI recipe"
    commit id: "Configure Qt5 integration"
    
    checkout main
    merge layer-development
    commit id: "Initial Build Test"
    
    branch ui-development
    checkout ui-development
    commit id: "QML Dashboard Design"
    commit id: "Touch Input Handler"
    commit id: "Vehicle Data Integration"
    
    checkout main
    merge ui-development
    commit id: "System Integration"
    
    branch optimization
    checkout optimization
    commit id: "Boot Time Optimization"
    commit id: "Memory Usage Tuning"
    commit id: "Display Performance"
    
    checkout main
    merge optimization
    commit id: "Release v1.0"
```

### Deployment Architecture
```mermaid
graph TB
    A[Development Environment] --> B[Build Process]
    B --> C[Generated Images]
    
    subgraph "Image Artifacts"
        D[autodisplay-image-raspberrypi4-64.wic]
        E[SDK for Development]
        F[Debug Symbols]
    end
    
    C --> D
    C --> E
    C --> F
    
    D --> G[SD Card Flashing]
    G --> H[Target Hardware Setup]
    
    subgraph "Target Deployment"
        I[Raspberry Pi 4B]
        J[7" DSI Touch Display]
        K[Power Supply 5V/3A]
        L[32GB SD Card]
    end
    
    H --> I
    H --> J
    H --> K
    H --> L
    
    I --> M[Boot & Initialize]
    M --> N[AutoDisplay Running]
```

### Performance Monitoring
```mermaid
graph TD
    A[System Monitoring] --> B[Performance Metrics]
    
    subgraph "Boot Performance"
        C[Boot Time: <15s]
        D[Service Startup: <5s]
        E[UI Ready: <20s total]
    end
    
    subgraph "Runtime Performance"
        F[CPU Usage: <50%]
        G[Memory Usage: <512MB]
        H[Display FPS: 30+]
        I[Touch Response: <100ms]
    end
    
    subgraph "System Health"
        J[Temperature Monitoring]
        K[Storage Usage]
        L[Network Connectivity]
    end
    
    B --> C
    B --> D
    B --> E
    B --> F
    B --> G
    B --> H
    B --> I
    B --> J
    B --> K
    B --> L
```

## Architecture Benefits

### Scalability
- **Modular Design**: Easy to add new vehicle features
- **Layer Architecture**: Reusable components for other projects
- **Service-Oriented**: Independent services for different functions

### Maintainability
- **Clear Separation**: UI, business logic, and data layers
- **Standard Frameworks**: Qt5/QML for automotive UI development
- **Documentation**: Comprehensive architecture documentation

### Performance
- **Optimized Boot**: Fast startup for automotive requirements
- **Efficient Rendering**: Hardware-accelerated graphics
- **Resource Management**: Controlled memory and CPU usage

### Industry Alignment
- **Automotive Standards**: Following automotive software practices
- **Yocto Best Practices**: Professional embedded Linux development
- **Modern UI Framework**: Qt5 widely used in automotive industry

---

> **Architecture Foundation**: This beginner project establishes the architectural patterns and development practices used throughout the portfolio, providing a solid foundation for more complex automotive systems. 