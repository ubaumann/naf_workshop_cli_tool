# Workshop FAQ: Leveraging Python Ecosystem for Quick CLI Tool Creation
**Proctor**: Urs Baumann  

---

!!! warning

    At the moment NAPALM does not support Python 3.13 ([PR](https://github.com/napalm-automation/napalm/pull/2137))

    Getting the path to a older Python version
    ```bash
    whereis python3.12
    ```

    Use the older version with Poetry
    ```bash
    poetry env use /usr/local/python/3.12.1/bin/python3
    ```

## General Information

### 1. What is this workshop about?
This workshop focuses on leveraging Python libraries like Nornir, Pydantic, Rich, and Typer to create powerful command-line interface (CLI) tools. Attendees will walk through hands-on exercises and real-world examples to build practical and user-friendly tools.

### 2. What experience level is required?
This workshop is suitable for all levels, from beginners to experts. Familiarity with Python basics is advised. The material will provide extra tasks for faster attendees.

---

## Pre-Workshop Preparation

### 3. What do I need to install before the workshop?

- **Python (version 3.10 or above)**
- **Python venv**: To avoid messing up your local Python setup, you should be able to create a virtual Python environment or work in a dedicated container/VM.
- **Unix**: It is recommended that you work on a Unix-based operation system like Linux or MacOS.
- **Libraries**: Poetry, Nornir, Pydantic, Rich, Typer, and others will be installed as part of the setup.
- **IDE**: You can use the IDE you desire. The proctor is most comfortable with VSCode.

### 4. Are there any pre-reading materials?
There is no mandatory pre-reading, but it does not harm to familiarize yourself with the official documentation of the libraries we will cover:

- [Poetry](https://python-poetry.org/docs/)
- [Nornir](https://nornir.readthedocs.io/)
- [Pydantic](https://docs.pydantic.dev/)
- [Rich](https://rich.readthedocs.io/)
- [Typer](https://typer.tiangolo.com/)

### 5. Do I need to bring any equipment?
Yes, please bring a laptop with Python and the necessary libraries pre-installed. Ensure that your laptop is configured with the appropriate permissions to install and run software.

### 6. Can I use GitHub Codespaces?
Yes. All the labs work well with GitHub Codespaces. Make sure your free quota is not exceeded. 

---

## Own Ideas

### 7. Can I implement my own ideas?
Definitely, it is appriciated to consider how you can apply the learned material to your specific use case. However, time is limited, and the proctor cannot focus on each idea in detail.

### 8. Lab access?
You can run your own lab on your computer or connect to your lab. The internet is shared to all attendees and therefore graphical transmissions are not recommended.

---

## Technical Questions

### 9. Will the workshop be hands-on?
Yes! This is a practical, hands-on workshop. You will be actively writing and running Python code to create CLI tools throughout the session.

---

## Additional Information

### 10. Who do I contact if I have questions before the workshop?
For any questions or concerns prior to the workshop, feel free to contact Urs Baumann directly through the Network Automation Forum channels.

### 11. What if I cannot keep up with the pace of the workshop?
The workshop is structured to accommodate various skill levels. If you fall behind, the proctor will be available to help, and you can use the break to catch up. The lab containes checkpoints with provided solutions.
