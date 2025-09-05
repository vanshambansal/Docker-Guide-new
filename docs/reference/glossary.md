# Docker Glossary

Comprehensive definitions of Docker terms and concepts. Use this glossary to understand Docker terminology and build your containerization vocabulary.

## Core Concepts

**Container**
: A lightweight, standalone, executable package that includes everything needed to run an application: code, runtime, system tools, system libraries and settings. Containers are isolated from each other and the host system.

**Image**
: A read-only template used to create containers. Images are built from a Dockerfile and can be stored in registries. Think of images as blueprints or recipes for containers.

**Dockerfile**
: A text file containing instructions for building a Docker image. Each instruction creates a new layer in the image.

**Layer**
: Each instruction in a Dockerfile creates a new layer. Layers are cached and reused across images to optimize storage and build times.

**Registry**
: A service for storing and distributing Docker images. Docker Hub is the default public registry, but private registries are also common.

**Repository**
: A collection of related Docker images, usually different versions of the same application. For example, `nginx` is a repository containing various nginx image versions.

**Tag**
: A label applied to images in a repository to distinguish different versions. Common tags include `latest`, `v1.0`, `stable`.

## Docker Architecture

**Docker Engine**
: The core Docker runtime that manages containers, images, builds, and more. Consists of a daemon, REST API, and CLI.

**Docker Daemon (dockerd)**
: The background service that manages Docker objects like images, containers, networks, and volumes.

**Docker Client**
: The command-line interface (CLI) that users interact with. Communicates with the Docker daemon via REST API.

**Docker Desktop**
: A GUI application for Windows and Mac that includes Docker Engine, Docker CLI, Docker Compose, and Kubernetes.

**BuildKit**
: Docker's improved build engine that provides better performance, caching, and advanced features for building images.

## Container Operations

**Detached Mode (-d)**
: Running a container in the background, returning control to the terminal immediately.

**Interactive Mode (-it)**
: Running a container with an interactive terminal session. The `-i` flag keeps STDIN open, `-t` allocates a pseudo-TTY.

**Port Mapping**
: Connecting a port on the host system to a port inside a container, allowing external access to containerized services.

**Volume Mounting**
: Attaching persistent storage to a container so data survives container restarts and removals.

**Environment Variables**
: Configuration values passed to containers at runtime, typically used for application configuration.

**Health Check**
: A test Docker runs to determine if a container is working correctly. Failed health checks can trigger container restarts.

## Networking

**Bridge Network**
: The default network driver. Creates a private internal network on the host where containers can communicate.

**Host Network**
: Container uses the host's networking directly. No network isolation between container and host.

**Overlay Network**
: Multi-host network allowing containers across different Docker hosts to communicate securely.

**Network Namespace**
: Linux kernel feature that provides network isolation between containers.

**Service Discovery**
: Mechanism for containers to find and communicate with each other using names instead of IP addresses.

## Storage

**Volume**
: A Docker-managed storage location that persists data beyond container lifecycle. Preferred method for persistent data.

**Bind Mount**
: Mounting a host file or directory into a container. The file/directory doesn't need to exist on Docker host beforehand.

**tmpfs Mount**
: Temporary filesystem stored in host memory. Data is lost when container stops.

**Storage Driver**
: Manages how images and containers are stored and managed on Docker host. Examples: overlay2, aufs, devicemapper.

## Building and Images

**Build Context**
: The set of files and directories sent to Docker daemon during image build. Defined by the path argument to `docker build`.

**Multi-stage Build**
: Dockerfile technique using multiple FROM statements to create optimized production images by separating build and runtime environments.

**Base Image**
: The starting point for building new images, specified in the FROM instruction of a Dockerfile.

**Scratch Image**
: An empty base image used for building minimal images, especially for static binaries.

**Distroless Image**
: Minimal base images containing only application dependencies, without package managers or shells.

**.dockerignore**
: File specifying which files and directories to exclude from the build context.

**Layer Caching**
: Docker's optimization that reuses unchanged layers across builds to speed up the build process.

## Orchestration

**Docker Compose**
: Tool for defining and running multi-container Docker applications using YAML configuration files.

**Docker Swarm**
: Docker's native clustering and orchestration solution for managing multiple Docker hosts.

**Service**
: In Docker Swarm, a service defines how containers should run across the cluster, including replicas, networks, and updates.

**Stack**
: A collection of services that make up an application, deployed and managed as a single unit.

**Node**
: A Docker host participating in a Docker Swarm cluster. Can be a manager or worker node.

## Security

**Namespace**
: Linux kernel feature providing process isolation. Docker uses namespaces to isolate containers from each other and the host.

**Control Groups (cgroups)**
: Linux kernel feature for limiting and monitoring resource usage of processes. Docker uses cgroups to control container resources.

**Security Context**
: Security-related settings applied to containers, including user ID, capabilities, and security profiles.

**Capabilities**
: Fine-grained privileges in Linux. Docker drops many capabilities by default to improve container security.

**AppArmor/SELinux**
: Mandatory Access Control systems that provide additional security layers for containers.

**Secrets Management**
: Secure storage and distribution of sensitive data like passwords, certificates, and API keys.

## Development and Operations

**DevOps**
: Development and Operations practices that Docker facilitates through consistent environments and deployment automation.

**CI/CD (Continuous Integration/Continuous Deployment)**
: Automated software development practices often implemented using containerized build and deployment pipelines.

**Infrastructure as Code (IaC)**
: Managing infrastructure through code and configuration files, often using containers for consistent deployments.

**Microservices**
: Architectural pattern where applications are built as a collection of small, independent services, often containerized.

**Blue-Green Deployment**
: Deployment strategy using two identical production environments (blue and green) to enable zero-downtime deployments.

**Rolling Update**
: Deployment strategy that gradually replaces old container versions with new ones to minimize downtime.

## Container Runtimes

**containerd**
: Industry-standard container runtime used by Docker and Kubernetes for low-level container operations.

**runc**
: Low-level container runtime that implements the Open Container Initiative (OCI) specification.

**CRI-O**
: Lightweight container runtime designed specifically for Kubernetes, implementing the Container Runtime Interface.

**OCI (Open Container Initiative)**
: Industry standards for container formats and runtimes to ensure compatibility across different platforms.

## Monitoring and Logging

**Container Logs**
: Output from applications running inside containers, accessible via `docker logs` command.

**Log Driver**
: Plugin that determines where container logs are sent (stdout, syslog, journald, etc.).

**Metrics**
: Quantitative measurements of container and application performance (CPU, memory, network usage).

**Observability**
: The practice of monitoring, logging, and tracing containerized applications to understand their behavior and performance.

**Telemetry**
: Automated collection and transmission of data from containers for monitoring and analysis.

## Registry and Distribution

**Docker Hub**
: Docker's public registry service for sharing container images. Contains official images and community contributions.

**Private Registry**
: Self-hosted or third-party registry for storing proprietary or internal container images.

**Image Manifest**
: JSON document describing image layers, configuration, and metadata.

**Content Trust**
: Docker's system for signing and verifying image integrity and publisher authenticity.

**Registry Mirror**
: Local copy of registry content to reduce bandwidth and improve pull performance.

## Performance and Optimization

**Resource Limits**
: Constraints on CPU, memory, and other resources that containers can consume.

**cgroups (Control Groups)**
: Linux kernel feature used by Docker to limit and monitor container resource usage.

**Copy-on-Write (CoW)**
: Storage optimization technique where containers share image layers until changes are made.

**Image Compression**
: Techniques to reduce image size for faster distribution and storage efficiency.

**Build Cache**
: Docker's mechanism for reusing previously built layers to speed up subsequent builds.

## Common Abbreviations

**CLI**
: Command Line Interface - the docker command and its subcommands

**API**
: Application Programming Interface - Docker's REST API for programmatic access

**JSON**
: JavaScript Object Notation - format used for Docker configuration and metadata

**YAML**
: YAML Ain't Markup Language - format commonly used for Docker Compose files

**TTY**
: Teletypewriter - terminal interface, used with `-t` flag for interactive containers

**STDIN/STDOUT/STDERR**
: Standard input, output, and error streams for container processes

**PID**
: Process ID - unique identifier for running processes

**UID/GID**
: User ID and Group ID - numeric identifiers for users and groups in Linux

**FQDN**
: Fully Qualified Domain Name - complete domain name specifying exact location in DNS hierarchy

This glossary provides essential Docker terminology for beginners through advanced users. Bookmark this page and refer to it when encountering unfamiliar terms in Docker documentation or discussions.