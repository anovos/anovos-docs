# Getting Started with Anovos

_Anovos_ provides data scientists and ML engineers with powerful and versatile tools for feature engineering.

To get you started quickly, we have prepared an interactive _Getting Started Guide_.
All you need is [Docker](https://www.docker.com) (see [here](https://www.docker.com/products/docker-desktop)
for instructions how to install it on your machine).

Then, you can launch the image as follows:
```bash
# clone the Git repository
git clone https://www.github.com/anovos/anovos.git
cd anovos/examples

# generate the Docker image (including Apache Spark and a Jupyter environment)
./create_anovos_examples_docker_image.sh

# Launch an anovos-examples Docker container
docker run -p 8888:8888 anovos-examples:latest
```

Note that calling `docker run` can require prefixing it with `sudo` on some machines.

To reach the Juypter environment, open the link to `http://127.0.0.1:8888/?token...` generated
by the Jupyter NotebookApp.
You can find the _Getting Started Guide_ in the `guides` folder within the Jupyter environment.
