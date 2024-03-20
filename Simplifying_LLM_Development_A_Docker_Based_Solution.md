# Simplifying LLM Development: A Docker-Based Solution

In an ever-changing tech landscape, mastering Python-based language model (LLM) projects can be challenging, especially for beginners. To address this, I've created a tutorial that introduces Docker, a tool that bridges the gap between Python Virtual Environments and Virtual Machines. Unlike traditional virtual environments like Venv, Conda, or Poetry that operate within the same OS runtime, Docker provides a complete runtime environment, including the OS and system libraries, separate from the host's kernel. This unique approach results in higher isolation and efficiency. Docker, being more comprehensive than virtual environments yet lighter than full virtual machines, is ideal for ensuring consistent and isolated development across different systems. This guide aims to simplify your development process and offer a clear pathway through the complexities of AI programming, whether you're a newcomer or a seasoned developer.

## **Initial Preparation: Setting up Python and Docker**

**1. Install Python:**

- **Special Note about Python Versions:** I recommend using Python version 3.10 or higher for AI/LLM work. While most AI/LLM projects on GitHub use later versions of Python, it's always best to check the project's repo for specific version recommendations.
- **Linux:** Python is often pre-installed. Check using `python --version`. If not installed, use your package manager (e.g., `sudo apt-get install python3`).
- **Windows and Mac:** Download and install Python from the [official Python website](https://www.python.org/downloads/).

**2. Install Docker:**

- **Linux:** Follow instructions from the [official Docker documentation](https://docs.docker.com/engine/install/).
- **Windows and Mac:** Download Docker Desktop from the [official Docker website](https://www.docker.com/products/docker-desktop) and follow the setup wizard.

### **Crafting a Custom Dockerfile for LLM Projects**

Docker offers the flexibility needed for LLM development. Below is a Dockerfile example for [pyautogen](https://github.com/microsoft/autogen/tree/main), adaptable for other LLM projects.

*Modify the Python version and base image as needed for your project. The provided example below will work for AutoGen but will require modification to work with other projects.*

```Dockerfile
FROM python:3.11-slim-bookworm

# Update and install dependencies
RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
        software-properties-common sudo\
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Setup a non-root user 'autogen' with sudo access
RUN adduser --disabled-password --gecos '' autogen
RUN adduser autogen sudo
RUN echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
USER autogen
WORKDIR /home/autogen

# Uncomment the lines below and create .env file in your docker directory.
# COPY .env /home/autogen/
# ENV OPENAI_API_KEY=$(cat /home/autogen/.env)

# Install Python packages
RUN pip install --upgrade pip

# AutoGen Specific Install
RUN pip install pyautogen[teachable,lmm,retrievechat,mathchat,blendsearch] autogenra
RUN pip install numpy pandas matplotlib seaborn scikit-learn requests urllib3 nltk pillow pytest beautifulsoup4

# Expose port
EXPOSE 8081

# Start Command
CMD ["/bin/bash"]
```

**Components to Retain or Modify for Different LLM Projects:**

1. **Base Image**: `python:3.11-slim-bookworm` is chosen for its lightweight nature and the pre-installed Python 3.11. This Debian-based image is popular in LLM development for its stable package repositories and community support, making it an excellent default choice. you can find a list of potential images on [DockerHub](https://hub.docker.com/)

2. **User Configuration**: The Dockerfile creates a non-root user `autogen` with sudo privileges. This is a security best practice, preventing the running container from having unrestricted root access, which could be a security risk.

3. **Python Environment**: The Dockerfile upgrades pip and installs a set of Python packages, including `pyautogen` with various options (`teachable`, `lmm`, `retrievechat`, `mathchat`, `blendsearch`) and `autogenra`. Depending on the specific LLM project you're working on, the packages will need to be adjusted. Always refer to the project's documentation for the required dependencies. This package selection is specific to the [AutoGen](https://github.com/microsoft/autogen/tree/main) project but can be adjusted for other LLMs. Always consult the specific LLM project's documentation for required dependencies.

4. **Environment Variable**: To securely handle the OpenAI API key, you can use a `.env` file in your Docker setup. This file will contain your API key, and Docker can use it without directly including it in the Dockerfile. Here's how you can implement this:

     a. **Create a .env File**: In your project directory, create a file named `.env`.

     ```plaintext
     OPENAI_API_KEY=your_actual_openai_api_key
     ```

     Replace `your_actual_openai_api_key` with your real OpenAI API key.

     b. **Modify Dockerfile**: Update your Dockerfile to copy the `.env` file into the container and set the environment variable:

     ```Dockerfile
     # Copy .env file containing the API key
     COPY .env /home/autogen/

     # Set environment variables from .env file
     ENV OPENAI_API_KEY=$(cat /home/autogen/.env)
     ```

     c. **Add .env to .gitignore**: To prevent accidentally pushing the `.env` file to a public repository, add `.env` to your `.gitignore` file:

     ```plaintext
     .env
     ```

     This method ensures that your API key remains secure and isn't included in the Dockerfile, which might be shared or committed to a public repository. It also keeps the key out of your Docker image's layers, adding an extra layer of security.

5. **Port Exposure**: The `EXPOSE 8081` command is indicative. You might need to expose different ports based on what your application requires.

6. **Start Command**: The `CMD ["/bin/bash"]` starts a bash shell. This can be changed to run your application directly, depending on your project's needs. we should provide an example of how to set this up.

### **Importing a Local Codebase into the Docker Container**

When working with Docker, you might have a local codebase that you want to use inside your Docker container. Docker provides a simple way to achieve this through the use of the `COPY` command in the Dockerfile or by mounting a volume when you run the container.

#### Option 1: Using the COPY Command in Dockerfile

You can modify the Dockerfile to include the `COPY` command, which copies files from your local file system into the Docker image. Here's an example:

```Dockerfile
# Copy local code to the container
COPY ./my_local_codebase /home/autogen/my_local_codebase
```

This line will copy the contents of the `my_local_codebase` directory from your local system to the `/home/autogen/my_local_codebase` directory inside the Docker container. Make sure to replace `./my_local_codebase` with the path to your actual local codebase.

#### Option 2: Mounting a Volume

For a more dynamic approach, especially if you anticipate making frequent changes to your local codebase, consider mounting a volume. This allows you to map a local directory to a directory inside the container, enabling real-time synchronization of files between your local environment and the Docker container.

Run the container with the following command to mount a volume:

```bash
sudo docker run -p 8080:8081 -v /path/to/local/codebase:/home/autogen/my_local_codebase -it --name {application_project_name} {image_name}
```

Replace `/path/to/local/codebase` with the path to your local codebase directory. This setup ensures that any changes made in the local directory are immediately reflected inside the container.

### **Advanced Docker Setups**

For more complex setups or specific requirements, Docker offers a variety of configurations and options. To delve deeper into Docker's capabilities, I recommend exploring the [official Docker documentation](https://docs.docker.com/). This resource provides comprehensive guides and explanations for various Docker functionalities, including networking, storage, security, and more.

### **Building and Running the Docker Container**

**1. Building the Docker Image:**

- Run in your Dockerfile directory:

     **Note**: the period at the end of the command is shorthand for Present working directory. If you want to have the image located somewhere else replace the "." with your directory desired patch

     ```bash
     docker build -t {image_name} .
     ```

**2. Running the Container:**

- Start and log into the container:

     ```bash
     docker run -it --name {application_project_name} {image_name}
     ```

     **Note**: if you need to map the exposed docker port to a local port on your device you can use this version of the command above. Note that the syntax is `-p <host_port>:<container_port>`

     ```bash
     docker run -p 8080:8081 -it --name {application_project_name} {image_name}
     ```

- Activate the Python virtual environment. Guess what you don't need to. This process removes the need to use a python virtual environment as you are already isolated from the local OS. If we had not created the Docker container user autogen then yes using a python environment would be the way to go for application security

     ```bash
     source /usr/src/app/autogen_env/bin/activate
     ```

**3. Saving the State of the Container:**

- To save the state of your container, use the `docker commit` command locally (not in the docker container). This creates a new image of the container's current state, which you can use to resume work later without losing your progress.

     ```bash
     docker commit {application_project_name} {new_image_name}
     ```

**4. Closing for the Day:**

- from inside of the container, exit the container:

     ```bash
     exit
     ```

- You should now be back in your local environment and you can stop the container:

     ```bash
     docker stop {application_project_name}
     ```

**5. Restarting Your Container:**

To restart your container after saving its state with `docker commit`, you can simply re-run the same commands 

**a. Start and log into the container:**

```bash
docker start {application_project_name}
```

**b. Attach to the running container:**

```bash
sudo docker exec -it {application_project_name} bash
```

This process will resume your container with the state and data preserved from when you last committed the changes. Remember, using `docker commit` before stopping your container is crucial for this process to work effectively.

### **Customizing for Different LLMs and Python Versions**

- **Change Python Version:** Modify lines with `python3.10` to your desired version.
- **Adapt for Different LLM:** Change `pip install pyautogen` to your LLM's package name.

### **Useful Docker Commands**

- View running containers:

  ```bash
  docker ps -a
  ```

- View Docker images:

  ```bash
  docker images
  ```

- Restart container setup:

  ```bash
  docker stop my_container
  docker rm my_container
  docker rmi my_image:latest
  ```

### **Conclusion**

Embracing D# Simplifying LLM Development: A Docker-Based Solution

In an ever-changing tech landscape, mastering Python-based language model (LLM) projects can be challenging, especially for beginners. To address this, I've created a tutorial that introduces Docker, a tool that bridges the gap between Python Virtual Environments and Virtual Machines. Unlike traditional virtual environments like Venv, Conda, or Poetry that operate within the same OS runtime, Docker provides a complete runtime environment, including the OS and system libraries, separate from the host's kernel. This unique approach results in higher isolation and efficiency. Docker, being more comprehensive than virtual environments yet lighter than full virtual machines, is ideal for ensuring consistent and isolated development across different systems. This guide aims to simplify your development process and offer a clear pathway through the complexities of AI programming, whether you're a newcomer or a seasoned developer.

