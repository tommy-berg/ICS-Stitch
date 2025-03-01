# MITRE Caldera™ with OT Plugin (Modbus) Docker Build

This repository builds a Docker image for MITRE Caldera™ with an added OT plugin (Modbus). The Dockerfile pulls the Caldera repository, sets up the environment inside Docker, and builds the application with additional plugins.

---

## Overview

MITRE Caldera™ is a cybersecurity platform designed to automate adversary emulation, assist manual red-teams, and automate incident response. It is built on the [MITRE ATT&CK™ framework](https://attack.mitre.org/) and is an active research project at MITRE.

The framework consists of two components:
- **Core System:** An asynchronous command-and-control (C2) server with a REST API and web interface.
- **Plugins:** Repositories that expand core capabilities by providing additional functionality, such as agents, reporting, and collections of TTPs.

This Docker build includes the OT plugin (Modbus) to extend Caldera’s functionality for operational technology scenarios.

---

## Security Notice

🚨 **Security Notice (17 Feb 2025 10:00 EST):**  
Please pull **v5.1.0** for a recent security patch for [CVE-2025-27364](https://github.com/mitre/caldera/issues/3138).  
**It is critical to update your Caldera instance, especially if you host it on a publicly accessible network.**  
For details, please refer to the vulnerability walkthrough linked above.

---

## Dockerfile Explanation

The provided `Dockerfile` performs the following steps:

1. **Base Image & Timezone Setup:**  
   Uses Ubuntu 24.04 as the base image and sets the timezone.

2. **Working Directory & Dependencies:**  
   Sets `/usr/src/app` as the working directory and installs necessary system packages such as Python 3, Git, curl, and Golang.

3. **Caldera Repository Clone:**  
   Clones the Caldera repository (branch 5.0.0) recursively.

4. **Plugin Verification:**  
   Checks for the existence of the Stockpile plugin (optional).

5. **Python Virtual Environment:**  
   Creates a Python virtual environment and installs the required Python packages from `requirements.txt`.

6. **OT Plugin Integration (Modbus):**  
   Clones the Modbus plugin repository into `plugins/modbus` and appends its configuration to the local configuration file (`conf/local.yml`).

7. **Additional Dependencies & Plugin Setup:**  
   Installs Go dependencies for the Sandcat plugin, updates agents, and sets up other plugins (Atomic, EMU, etc.).

8. **VueJS Front-End Build:**  
   Installs Node.js and npm to build the VueJS front-end for Caldera, then cleans up unnecessary packages.

9. **Expose Ports:**  
   Exposes ports for HTTP, HTTPS, contact ports, WebSockets, DNS tunneling, SSH tunneling, and FTP C2 channels.

10. **Entry Point:**  
    The image starts the server using:
    ```bash
    python -X dev server.py
    ```

---

## Build & Run Instructions

### Build the Docker Image

To build the Docker image without using the cache, run:

```bash
docker build --no-cache -t caldera-modbus .
```

### Run the Docker Container
To run the container (mapping port 8888 inside the container to port 80 on your host), use:

```bash
docker run -d -p 80:8888 --name caldera_modbus caldera-modbus
```
Note: The application listens on port 8888 inside the container, even though it is mapped to port 80 on the host.

### Known Issue: YARL URL Parsing Bug
There is a known bug in the YARL component that prohibits hosting Caldera on ports other than 8888. For more details, please refer to the GitHub issue #3138.

### Additional Information
For more details on MITRE Caldera™, refer to the official repository and documentation provided by MITRE.

